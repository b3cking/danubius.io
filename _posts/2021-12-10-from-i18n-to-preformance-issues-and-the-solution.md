---
layout: post
author: Bence Ignácz
title: From i18n to performance issues and the solution
---
{% include image.html src="/images/mspecs_permormance_tuning.jpg" alt="Man is loosing it with head on his computer" %}

When you start to plan the pillars for your application one topic should be the language support. If you don't want to support multiple language, it's very simple, every text are staticly stored and you don't need to care about it.
But if you want to support multiple regions and languages you need to review some questions: how to store the static texts? How should I store dynamic content? There are a lot of solution to translate static content on the internet, but the second question is more intresting.
One of our customer decided to store them in a relational database. This is not a worst idea, but not the best as well. When you store the multilangual content in relations, you need careful and very precise design your entities becaus it can be a pitfall very soon.

<!--more-->

But step back a bit first. The original requirement was to find the bottleneck in the system and do a performance tuning. 

### The situation

The application stack is very forward-looking:
* Azure hosting
* PostgreSQL database
* Elastic Search
* Microservices architecture by Quarkus framework
* Microfrontend architecutre with Angular
* Keycloak as authentication backend

The data between the relational database and indexed Elastic schema is managed by Hibernate search. There were two problematic points: the first is the re-indexing process, which runs for more than 30 mins, the second one shows item details on the frontend, it runs for 3 seconds!

That was clear on the first look, the core of the problem was somewhere in the architecture of the application not in the infrastructure. The amount of data is not too much, the services in the cloud are well designed.

### Investigation

So we've started to deep dive into the backend first. The entities were suspicious at first. There are a lot of relations between the entities and all of them are defined as eager fetching. It can be a bottleneck, to load all of them with a single entity loading. To check the SQL execution, we've set the JPA to show SQL statements and that was very shocking. When the frontend get item details, the Hibernate generated a 40 000 line query.

After looking at this scary behavior, that was really clear, we need to do some refactoring. But where should we start? 
Before doing any refactor, the following things must inspect before starting it:
* When I do the refactor, which layers will affect it?
* What are the side effects?
* Should I change the technology or parts of the architecture?

Keeping these aspects in mind we've started the thinking. The first idea was to change the relation fetching from eager to lazy, but this is very dangerous because we need to do a whole regression test and fix the queries where necessary. The second one is to build a well-managed cache layer. It is a bit better, but we had time pressure due to the fixed release date.

After an investigation of the entity structure, we found the problematic relationship with the multilanguage content. They are stored over 2 entities:

Item 1-1 MultiLanguageString 1-* LanguageString

Since most properties of an item are multilingual this relation is very painful to load. To resolve this problem we decided to store this data in another way. As we have seen, the frontend got this data in a very simple structure and there is no other operation on the backend side. 
From this point the solution is very simple: store every translatable content in JSON format in the database. This is supported by most database engines.
To do this, we have created schema migration scripts and implemented our own `TypeBinder` for Hibernate.

### Data migration

The schema migration is managed by Liquibase, it is a very powerful migration tool. First, we have renamed the old column that stored the reference to the MultiLanguageString, e.g.: `name_old`. Then a new `name` column has been created with `json` type. This is a sample statement:

```sql
alter table item
    add column name json;
```

The harder part came after that: move the existing data to the new column in json structure.
The expected structure is the following:

```json
{
	"languages": [{
			"content": "Item name (NL)",
			"language": "nl"
		},
		{
			"content": "Item name (EN)",
			"language": "en"
		},
		{
			"content": "Item name (FR)",
			"language": "fr"
		}
        ]
}
```

To build the below structure we've used the Postges JSON functions with SELECT statements. It's a bit tricky:

```sql
update item i
set name = (
    select json_build_object('languages', (
        select json_agg(p)
        from (
                 select ls.content, ls.language
                 from item i2
                          left join multilanguage_string ms on i2.old_name = ms.id
                          left join language_string ls on ms.id = ls.multilanguage_string
                 where i2.id = i.id
             ) p
    ))
);
```
First, we create the wrapper object with on `languages` property. To do this we have used the `json_build_object` function, where the first parameter is the property name and the second is the property value. Now the second parameter is a select statement whose result is aggregated into an array with the `json_agg` function.

### Backend refactor

Unfortunately, the JSON datatype is not supported by Hibarenate out of the box. First, we need to register the new type on the database dialect. There is a lot of ways to do this, in our case we've extended the PostgresDialect:

```java
public class ExtendedPostgresDialect extends PostgreSQL10Dialect {

    public ExtendedPostgresDialect() {
        super();
        this.registerColumnType(Types.JAVA_OBJECT, "json");
    }
}
```

By these few lines of code, the driver can identify the Postgre `json` type. Now we need to care about the entity handle mechanism for Hibernate. First, change the entity property from the relation to a simple POJO. The POJO looks like this:

**MultiLanguageString**

```java
public class MultiLanguageString {

    public List<LanguageString> languages;

    (...getters)
    (...setters)
}
```

**LanguageString**

```java
public class LanguageString {

    public String content;
    public String language;

    (...getters)
    (...setters)
}
```

Now we can change the entity property type from `MultiLanguageStringEntitiy` to `MultiLanguageString`. But something missing... We don't tell to Hibernate, this is a JSON type column.

We need to create our SQL type for Hibernate:

```java
public class MultiLanguageStringType implements UserType
```

Since we implement the `UserType` we need to implement some methods. The most important parts are the `nullSafeGet` what it reads the record from the database and the `nullSafeSet` which writes the data to the column.

Here are two sample implementations of these methods:

**`nullSafeGet`**

```java
@Override
public Object nullSafeGet(ResultSet resultSet, String[] names,
        SharedSessionContractImplementor sharedSessionContractImplementor, Object o)
        throws HibernateException, SQLException {
    final String cellContent = resultSet.getString(names[0]);
    if (cellContent == null) {
        return null;
    }
    try {
        final ObjectMapper mapper = new ObjectMapper();
        return mapper.readValue(cellContent.getBytes(StandardCharsets.UTF_8), returnedClass());
    } catch (IOException e) {
        throw new RuntimeException("Failed to convert string to json: " + e.getMessage(), e);
    }
}
```

**`nullSafeSet`**

```java
@Override
public void nullSafeSet(PreparedStatement preparedStatement, Object value, int i,
        SharedSessionContractImplementor sharedSessionContractImplementor) throws HibernateException, SQLException {
    if (value == null) {
        preparedStatement.setNull(i, Types.NULL);
        return;
    }
    try {
        final ObjectMapper mapper = new ObjectMapper();
        final StringWriter writer = new StringWriter();
        mapper.writeValue(writer, value);
        writer.flush();
        PGobject pGobject = new PGobject();
        pGobject.setType("json");
        pGobject.setValue(writer.toString());
        preparedStatement.setObject(i, pGobject, Types.OTHER);
    } catch (IOException e) {
        throw new RuntimeException("Failed to convert json to string: " + e.getMessage(), e);
    }
}
```

The other important part of the type implementation is the `package-info.java`. Here you can specify your own types, for example:

```java
@org.hibernate.annotations.TypeDef(name = "MultiLanguageStringType", typeClass = MultiLanguageStringType.class)
package com.danubiusinfo.sample.type;
```

After the type definition, we can use it in our entity. With the `@Type` annotation define the custom type of the column. For the correct validation use the `columnDefinition` attribute in the `@Column` annotation as well.

```java
@Type(type = "be.mediaspecs.mediadb.language.type.MultiLanguageStringType")
@Column(name = "name", columnDefinition = "json")
public MultiLanguageString name;
```

That's it, from now we are able to store translation data in the relational database as JSON objects. By this refactor the reindexing process was reduced to ~3 minutes, the item loading on the frontend to ~950ms.

Instead of doing a multi-tier refactor we changed only the data layer and the ORM layer a bit. After this solution, our customer decided to continue the development with us and he entrusted us with even more development.
