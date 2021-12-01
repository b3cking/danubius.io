---
layout: post
author: Istvan Kis
title: How can feature flagging help you during release and customer experiments?
keywords: feaute flag management, feature flagging, what is feature flagging, feature flags in software development
---
{% include image.html src="/images/feature-flag0.webp" alt="On and off button" %}

Have you ever been working in a situation as a project manager, business owner, **product owner** on a project or in an organisation, when a major change you have been waited for so long was deployed to production related to an application you were responsible for, and what worked perfectly yesterday’s now broken, but you couldn't intervene? Are you familiar with the terms rollback, while you shouldn’t have to? Perhaps you have developed new functions with IT guys, but when you went live, you couldn't make sure everything was functioning properly, before opening it to the full audience of customers?

Have you ever done market segmentation as a **marketing expert**, but it never occurred to you that the app could be used to target specific users, and display differently to some groups than to others? Did you want to try something new, but weren't sure if it would have the same effect that you expected? Has IT supported A/B testing so far?

As a **developer or operator**, have you ever not slept the night after a release, because it was impossible to turn off non-operable features, systems were slow to revert to previous state, or there were interdependencies between pieces of code that only a full revert helped and lost all new features? Have you missed proper testing ever? Has it taken days to do a sweaty merge after months of development on several different branches even after following some well-written strategy like Gitflow?

Feature flagging is not the Holy Grail, but it can be very helpful for all involved stakeholders in the software development life cycle.

<!--more-->

## What is feature flagging?

There are many definitions out there, but probably the most precise one if we say that feature flagging is an opportunity to change the application’s behaviour and feature sets after the code has been deployed to the running environment, e.g. production one. This leads to many other great chances during and after the standard development process.

{% include image.html src="/images/feature-flag.png" alt="Feature flag path - show or hide" %}

## Main benefits of feature flagging

There are basically 3 (maybe 4) important categories, where this mindset and strategy can bring in great values.

### Release management

Probably the most used use-case. With feature flag approach changes can focus on smaller segments of customers – why would you roll-out the product to the entire audience after weeks or months of tests, while you can push features to production and receive feedback from only a chunk of people and then iteratively increase the userbase? They can be even more targeted, such as enabling risky functions only for the patient ones at first. If something is not working, one click, and the new features become unavailable. It is easy to manage and no need for any IT involvement to do that. (*Further goods here: setting the exact timing and audience of the release, iterative rollout, friends and family, highly decreased release risk, beta/canary release, etc.*)

### Development and Operation after deployment

It is not enough to have a successful release, systems need to be kept up and running. In the most cases feature flagging is easy to implement from development point of view, operators also like to work with, because even though organisations using feature flagging tend to release more and more frequently over time, this still doesn’t cause more rollbacks from production, quite the contrary: the rollbacks’ number decrease due to the fact, that switching off a feature set does not mean new deploy or code change at all. Even later, if a team responsible for a certain application finds, that some elements of the infrastructure or a component of a software stack is not functioning, which causes performance issues, they have a chance to intervene immediately and switch off high resource–consuming features. *(We do not cover topics like software architecture design and high-availability - hereinafter HA - here, obviously feature flagging cannot replace all these steps later, if they were skipped earlier. Further goods here: tweaking on-the-go, direct access from business PoV to operation (need to handle with care), no rollback, etc.*)

### Experimenting with real customers

Marketing or Sales can work with feature flags to get more insights about their customers built on top of the data company alerady owns. Let’s imagine they want to show a different advertisement picture to people above and under 30 years, but they do not know how long this promotion or the differentiation will last. With feature flags the problem can be solved easily, and after development and deploy, no IT assistance need to change the features, targets, scopes, etc.

{% include image.html src="/images/feature-flag2.png" alt="A/B testing switch" %}

### “Functionalities”

Some features can be available only for a smaller set of customers by design, and feature flag SaaS solutions can be part of the design. It is not recommended, but a real option.

## Possible threats

### Security

Never ever publish a possibly vulnerable, not properly tested feature to production even with switched off flag, because it is very easy to miss-click, allow it accidentally, and provide an easy-to-break honeypot and optional attack vectors to possible attackers.

### Consistency

While more and more features go to production with the approach mentioned, later all these can behave as a technical debt, which obviously has a price. Do not forget about it.

On the other hand, feature flagging **strategy** is good to have at start, because later it can be difficult to introduce. Consider defining process, controlling access of features, audit of switching on and off, maintain documentation, guides and set expectations from feature flagging and **meisure** to have a chance to improve.

### Availability

What happens if feature flag service is not available? Answer this questions and introduce local cache, especially if you use SaaS flagging system, but it can be beneficial also, if you have your own flagging application, because this way you do not need to build or buy a HA infrastructure for your internal feature flagging solution. Latency and tolerating failures are also worth thinking about.

## Cost

In general, it is a cheap solution compared to other IT techs, but it depends on how you develop the environment: do you ask internal devs to provide internal feature flagging system, or pay for a SaaS. Whichever it is, usually the cost/income ratio is quite good.

## Where to start

We will use [ConfigCat](https://www.configcat.com) in the example, but any other SaaS provider will work very similar way. First of all, register on the webpage and set our first feature.

{% include image.html src="/images/feature-flag3.png" alt="Target specific user" %}

To connect the API to our local code, follow the steps described on the FAQ. As we use Python right now, install configcat-client.

```shell
pip install configcat-client
```

Use your favourite IDE, and try it out.

<iframe src="https://pastebin.com/embed_iframe/13A2DP52" style="border:none;width:100%;height:210px"></iframe>

If this works, the good old if-else statements do the rest of the magic, and marketing specialist, business owners can switch features on and off from now on by a single click.

### Example opportunities

#### Iterative rollout, A/B testing

Start with like 5% of the customers, and if everything seems fine, increase again and again.

{% include image.html src="/images/feature-flag4.png" alt="Iterative rollout" %}

#### Geolocation based appearance

Feature 4 is so great, and only Germany can reach.

{% include image.html src="/images/feature-flag5.png" alt="Country targeted appearance" %}

## Some providers on the market (SaaS too)

- [ConfigCat](https://www.configcat.com) (we use this one)
- [Featureflags.io](https://featureflags.io) (Opensource)
- [LaunchDarkly](https://launchdarkly.com)
- [Vwo](https://vwo.com)
- [Cloudbees](https://www.cloudbees.com)
- [Hackle](https://hackle.io/)
- [Split.io](https://www.split.io)

## Summary

Feature flagging is a key element of giant tech companies and using it may help your business and team grow, transform to a fast-paced environment, where release, testing and experimenting are not fearful words, but something you are familiar with from the daily routine.

In [Danubius](https://danubiusinfo.com/), we used the approach in different domains (education, banking, and many local product development), and in different countries (Belgium, The Netherlands, Hungary, UK, etc.), feature flagging was a great help for every projects and every customers.
