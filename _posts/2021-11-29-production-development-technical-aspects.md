---
layout: post
author: Istvan Kis
title: Startup product development risks from technical aspects
---
![_config.yml]({{ site.baseurl }}/images/startup-1.png)

Do you **have the perfect product idea**, a unique and innovative approach, know your potential competitors, and are convinced you can do better? Do you have ideas about how to break into the market, have ideas about marketing and sales, **but IT development is not your thing**? You're not alone, we've recently been talking to a number of similar startup owners and entrepreneurs across Europe, some of whom we've helped to turn their ideas into reality from a development perspective. Many think that they can cover the development part of the process of bringing a product to market by hiring one or two (up to four or five, maybe more) programmers and hope that they will answer all the technical questions. Even if the best developers in the world have been recruited for the task, it will not be that simple and smooth, there are many pitfallss that can hinder the market launch, which can be avoided to a very large extent by conscious planning and preparation. In this post, we will try to gather some of the key, mainly IT-related aspects that are typically missing from business visions and that, apart from writing the source code, will be necessary for the IT and hence business launch of a successful product.

<!--more-->

## What are your predefined goals?

Experience has shown that almost everyone wants three things: to produce the software, the tool, the end product **quickly**, with good **quality** and **cheaply**. The bad news is that we can't pull all the sliders to the maximum, because these three attributes are closely interrelated. But there is also good news: all three attributes are also completely relative, and you may not think of "*cheap*" as the same as Apple, or "*fast*" as the same as Google. Of course, you can have the same goals for a startup as you do for a BigTech company, but that's usually not the case, and even with a more modest budget, smaller hardware or cloud subscription, slower response time, etc. you can still serve your customers very well with maximum satisfaction. Okay, but then where is the limit, you may ask. How do you know what you technically need, and really need? Engineering is also about coordinating these aspects, and effectively **making trade-offs** along the way.

## Roles, or what's beyond the programmer

### Business and System Analyst

It helps gather your business needs and expectations, **translates** them into requirements, and ultimately into "IT language" so that developers can work from a detailed specification, making their work much faster and more efficient, and significantly reducing lead times.

### Software Engineer / Developer

The traditional developer role, you probably know it best, may overlap with the Analyst on design questions, and then the actual software **code** is written by the Developer. They do frontend (so the layout have visibility and buttons), backend (they make the data persistent) and interface development, and optimally try to deliver the best quality work available. That's fine, but because we are human, we make mistakes, and we need to be aware of that, and realise it before actual customers.

### Manual and Automatic Test Engineer

You may think that you have business people to do the testing, and that's perfectly fine, but in today's modern software development, especially for complex applications, it may be worth doing some of these automatically, or supplementing manual testing. We won't go into this in detail, but in general, the better quality product you want to develop, the more coordinated and system-wide testing you need to do (from the bottom of the testing pyramid: unit tests, module and component tests, integration tests, interface and end-user tests). Setting up quality gates, defining and enforcing a set of criteria is one of the most important **guarantees of quality**, but you may not need all layers of it.

### Additional roles

In addition to the above, depending on the methodology you are working with (typically agile, less often waterfall or other), you may need other roles: **Scrum Master** / PM (responsible for team operations), **Product Owner** (responsible for requirements specification, prioritisation and decisions - typically available on the client side), **DevOps Engineer** (typically responsible for fast and automated delivery, monitoring and live operations, logging), and so on. These are also needed, but to what extent and for how long also depends on the scope of the current project and the requirements.

![_config.yml]({{ site.baseurl }}/images/startup-2.jpg)

## I heard good things about *this and that*

### “Be agile/cloud/microservice, because it's fast/trendy/sexy"

Indeed, agile can be great, cloud can be scalable, and microservice architecture offers a lot of possibilities (it's okay if we don't understand what it is at the moment), but not because we have Dumbledore's wand in our hands, but because e.g. agile methodology is based on the simplest possible elements, cloud is only somebody else’s computer with all the advantages and disadvantages, etc.

Typically, there is a lot of misunderstanding that agile can be good for everything, because "*that's why it's agile*" - not necessarily. This is not supposed to be a detailed description of agile, but I would like to highlight a few points, not exclusively about the methodology, which we need to consider:

- In sprints, **scope** (backlog items) **needs to be managed, planned, measured back**, so that – as the team improve together gradually –, we can iteratively increase the amount of work done during an iteration.

- The remaining tasks need to be prioritised, **refined**, so that during planning we know what is next in the queue, so that we can work even more efficiently and quickly.

- The tasks (stories) should be clear for everyone and have an attribute about when those **are considered completed** (Definition of Done, Acceptance Criteria), and testers should pay attention to these, writing test cases and scenarios based on them.

- The software runs in **different environments** and release management levels typically, developers work and test in a development environment, business people test in some kind of acceptance environment (QA, UAT, etc.), in between there may be different integration tests. It is important that the quality gates mentioned above are defined, as this is how we can eliminate any bugs that are left in by accident or mistake. The various non-functional updates, patches, system fixes, etc. need to be tested somewhere if live operation is not to be compromised. These, and the quality gates between them, need to be **defined**, **developed** and **operated**.

- Above we have mentioned functional tests, but we would not like to see the application slow down or become inaccessible due to higher load during live operation, or even worse, a malicious hacker breaking into our system. This would require **additional tests**, performance and security tests (penetration).

- Fortunately, it is more and more discussed nowadays, but even if your application works functionally fine, if nobody likes to use it because the interface is not intuitive, not logical, etc., its value is kind of low. It is essential to involve a **UX/UI** expert and/or **knowledge** at the design stage.

- In terms of **parallelisation** and **scheduling**, it is important to see at which points we can accelerate throughput by bringing in additional resources and/or teammates based on **dependency**-graph. A well-known examples is that 9 women cannot give birth to a baby in one month.

- Even for a cohesive team, meeting the **end deadline** can be a challenge in agile operations, as the team is constantly producing value through continuous feature development, but this does not mean that, without proper planning, all this value produced in small sprints will actually add up to the feature set needed to meet the end goal in X months or years.

![_config.yml]({{ site.baseurl }}/images/startup-3.png)

### "Let's work, don't waste time on daily standup and planning, it's agile, we'll improve by definition“

We've heard phrases like this many times over the years, and none of our clients would say it today. They have come to understand that these **ceremonies** are part of the job. Talking through priorities and planning steps is an integral part of the work, as is looking at what's been completed, how the team improved at the end of an iteration. Then and there, the fact that - let’s say - *Peter*'s task delayed a bit because he was waiting for *George* may not seem important at first, but if we can figure out why the dependency was not handled properly, we can be even better and faster in the next iteration, and the associated developer frustration can be also reduced or in lucky situations completely eliminated.

## Summary

The above was not intended to describe a complete software development process, we did not want to give a coherent summary, but to show that product development is (unfortunately) not so simple in IT terms that it can be fully covered by one or two persons with only programming skills. It **requires a bigger picture** of where we want to go from where, in what timeframe, how much money we have available, what our expectations are, and keeping priorities and dependencies in check to move forward. It is true for agile as well.

If you think you would like to discuss any of your development project details with Danubius staff, we are available on any of our [contact details](https://danubius.io/about), and more than happy to help you out.
