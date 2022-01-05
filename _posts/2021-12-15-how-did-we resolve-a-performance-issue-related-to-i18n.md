---
layout: post
author: Bence Ignácz
title: How did we resolve a performance issue related to i18n?
---
{% include image.html src="/images/mspecs_permormance_tuning.jpg" alt="MacBook is opening in fancy lights" %}

When you start to plan the pillars of your application, one topic you should consider is region support. If you don't want to support multiple languages, it's very simple, every text can be statically stored, and you don't need to care about it later. But what if you want to support multiple regions and languages? Before implementation, you need to answer some related questions: How to store the static texts? What about dynamic content? There are many solutions on the internet for the first part, but the second one is much more interesting. One of our customers decided to handle all these with a relational database. This is not the worst idea, but not the best either. When you store the multilingual content in relations, you need careful and very precise design of your entities, because it can become a pitfall very soon.

<!--more-->

Step back a bit first. The original requirement was to find the bottlenecks in the system, and do a performance tuning.

## The situation

The application stack is very forward-looking:
* Microsoft Azure cloud hosting
* PostgreSQL relational database
* ElasticSearch
* Microservices architecture by Quarkus framework
* Microfrontend architecture with Angular
* Keycloak as authentication backend

The data between the relational database and the indexed Elastic schema is managed by Hibernate search. There were two problematic points: the first was the re-indexing process, which ran for more than 30 minutes, the second one showed item details on the frontend, and it ran for 3 seconds, while the user kept waiting!

That was clear on the first look, the core problem was somewhere in the architecture of the application not in the infrastructure. The amount of the data was not too much, the services in the cloud were well designed.

## Investigation

So we've started to deep dive into the backend first. The entities were suspicious at first sight. There were a lot of relations between the entities and all of them are defined as eager fetching. It can be a bottleneck, to load all of them with a single entity loading. To check the SQL execution, we set JPA to show SQL statements and that was very shocking. When the frontend got item details, the Hibernate generated a **40 000 line query**.

After looking at this scary behavior, that was really clear, we needed to do some refactoring. But where should we start? Before starting any refactor, the following things must be inspected:
* When I do the refactor, which layers will be affected?
* Will there be any side effects?
* Should I change the technology or parts of the architecture?

Keeping these aspects in mind we've started thinking about possible solutions. The first idea was to change the relation fetching from eager to lazy, but this could be very dangerous, because we needed to do a whole regression test and fix the queries, where it was necessary. The second one was to build a well-managed cache layer. It was a bit better, but we had time pressure due to the fixed release date, so we have discarded this option as well.

After an investigation of the entity structure, we found the problematic relationship with the multilanguage content. They were stored over 2 entities:

{% include image.html src="/images/mspecs_entity_relation.jpg" alt="MSpecs entity relation diagram" %}

Since most properties of an item are multilingual, this relation is very painful to load. To resolve this problem we decided to store this data in another way. As we saw, the frontend received this data in a very simple structure, and there was no other operation on the backend side. From this point the solution was very simple: store every translatable content in JSON format in the database. This is supported by most database engines. To do this, we have created schema migration scripts and implemented our own `TypeBinder` for Hibernate.

## Data migration

The schema migration was managed by Liquibase, it proved to be a very powerful migration tool. At first, we renamed the old column that stored the reference to the MultiLanguageString, e.g.: `name_old`. Then a new `name` column has been created with `json` type. This is a sample statement:

```sql
alter table item
add column name json;
```

The harder part came after that: move the existing data to the new column in JSON structure. The expected structure was the following:

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

To build the structure below, we've used the Postges JSON functions with SELECT statements. It's a bit tricky:

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
At first, we created the wrapper object on `languages` property. To do this we used the `json_build_object` function, where the first parameter was the property name and the second was the property value. Now the second parameter was a select statement, whose result was aggregated into an array with the `json_agg` function.

## Backend refactor

Unfortunately, the JSON datatype is not supported by Hibarenate by default. First, we needed to register the new type on the database dialect. There are many ways to do this, in our case we extended the PostgresDialect:

```java
public class ExtendedPostgresDialect extends PostgreSQL10Dialect {

    public ExtendedPostgresDialect() {
        super();
        this.registerColumnType(Types.JAVA_OBJECT, "json");
    }
}
```

By these few lines of code, the driver could identify the Postgres `json` type. After we needed to care about the entity handle mechanism for Hibernate. At first, changed the entity property from the relation to a simple POJO. The POJO looked like this:

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

After we could change the entity property type from `MultiLanguageStringEntitiy` to `MultiLanguageString`. But something was missing... We didn't tell the Hibernate that this was a JSON type column.

We needed to create our SQL type for Hibernate:

```java
public class MultiLanguageStringType implements UserType
```

Since we implemented the `UserType`, we also needed to implement some additional methods. The most important parts were the `nullSafeGet`, which read records from the database and the `nullSafeSet`, which wrote the data to the column.

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

The other important part of the type implementation was the `package-info.java`. Here you can specify your own types, for example:

```java
@org.hibernate.annotations.TypeDef(name = "MultiLanguageStringType", typeClass = MultiLanguageStringType.class)
package com.danubiusinfo.sample.type;
```

After the type definition, we could use it in our entity. With the `@Type` annotation custom type of the column can be defined. For the correct validation use the `columnDefinition` attribute in the `@Column` annotation as well.

```java
@Type(type = "be.mediaspecs.mediadb.language.type.MultiLanguageStringType")
@Column(name = "name", columnDefinition = "json")
public MultiLanguageString name;
```

## Summary

From now on, we were able to store translation data in the relational database as JSON objects. By this refactor the reindexing process was reduced **from ~30 minutes to ~3 minutes**, the item loading on the frontend **from ~3 seconds to ~950 milliseconds**.

Instead of doing a multi-tier refactor we changed the data layer only, and the ORM layer a bit. After this solution, our customer decided to continue the development with us, and he entrusted us with even more development.

If you are interested in the code more deeply, you can find our example in the  [Hibernate JSON datatype using Quarkus framework](https://github.com/danubiusinfo/java-samples/tree/master/quarkus-hibernate-json)
repository.
