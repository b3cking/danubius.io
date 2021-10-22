---
layout: post
author: Istvan Kis
title: 🇬🇧 How can feature flagging help you during release and customer experiments?
---

Have you ever been working in a situation as a project manager, business owner, **product owner** on a project or in an organisation, when a major change you have been waited for so long was deployed to production related to an application you were responsible for, and what worked perfectly yesterday’s now broken, but you couldn't intervene? Are you familiar with the terms rollback, while you shouldn’t have to? Perhaps you have developed new functions with IT guys, but when you went live, you couldn't make sure everything was OK before opening it to the full audience of customers?

Have you ever done market segmentation as a **marketing expert**, but it never occurred to you that the app could be used to target specific users, and display differently to some groups than to others? Did you want to try something new, but weren't sure if it would work? Has IT supported A/B testing so far?

As a **developer or operator**, have you ever not slept the night after a release, because it was impossible to turn off non-operable features, systems were slow to revert to previous state, or there were interdependencies between pieces of code that only a full revert helped? Has it taken days to do a sweaty merge after months of development on several different branches even with some well-written strategy like Gitflow?

Feature flagging is not the Holy Grail, but it can be very helpful to all involved stakeholders in the SDLC.

<!--more-->

## What is feature flagging?

There are many definitions out there, but probably the most precise one if we say that feature flagging is an opportunity to change the application’s behaviour and feature sets after the code has been deployed to the running environment, e.g. production one. This leads to many other great chances during and after the standard development process.

![_config.yml]({{ site.baseurl }}/images/feature-flag.png)

## Main benefits of feature flagging

There are basically 3 (maybe 4) important categories, where this mindset and strategy can bring in great values.

### Release management

Probably the most used use-case. With feature flag approach changes can focus on smaller segments of customers – why would you roll-out the product to the entire audience after weeks or months of tests, while you can push to production and receive feedback from only a chunk of people and then iteratively increase the user base? They can be even more targeted, such as enable risky functions only for the patient ones at first. If something is not working, one click, and the new features become unavailable. It is easy to manage and no need for any IT involvement to do that. (Further goods here: setting the exact timing and audience of the release, iterative rollout, friends and family, highly decreased release risk, beta/canary release, etc.)

### Development and Operation after deployment

It is not enough to have a successful release, systems need to be kept up and running. In the most cases feature flagging is easy to implement from development point of view, operators also like to work with, because even though organisations using feature flagging tend to release more and more frequently over time, this still doesn’t cause more rollbacks from production, quite the contrary: the rollbacks’ number decrease due to the fact, that switching off a feature set does not mean new deploy or code change at all. Even later, if a team responsible for a certain application finds, that some elements of the infrastructure or a component of a software stack is not functioning, which causes performance issues, they have a chance to intervene immediately and switch off high-consuming features. *We do not cover topics like software architecture design and high-availability here, obviously feature flagging cannot replace all these steps later, if they are missing.* (Further goods here: tweaking on-the-go, direct access from business PoV to operation (need to handle with care), no rollback, etc.)

### Experimenting with real customers

Marketing or Sales can work with feature flags to get more insights about their customers based on any data company owns. Let’s imagine we want to show an advertisement picture to people above and under 30 years, but we do not know how long. With feature flags this can be solved easily, and after development and deploy, no IT assistance need to change these features, targets, scopes, etc.

![_config.yml]({{ site.baseurl }}/images/feature-flag2.tif)

### “Functionalities”

Some features can be available only for a smaller set of customers by design, and feature flag SaaS solutions can be part of it. It is not recommended, but a real option.

## Possible threats

### Security


Never ever publish a possibly vulnerable, not properly tested feature to production even with switched off flag, because it is very easy to miss-click, allow it, and provide an easy-to-break honeypot and optional attack vectors to possible attackers.

### Consistency

While more and more features go to production with this approach, this can behave as a technical debt, which obviously has a price. Do not forget about it.

On the other hand, feature flagging **strategy** is good to have at start, because later it can be difficult to introduce. Consider process, controlling access of features, switching on and off audit, maintain documentation, guides and define expectations.

### Availability

What happens if Feature Flag Service is not available? Introduce local cache, especially if you use SaaS flagging system, but it can be beneficial, if you have your own flagging application, because this way you do not need to build or buy a HA infrastructure for your internal feature flagging solution. Latency and tolerating failures are also good to think about.

## Cost

In general, it is a cheap solution compared to other IT techs, but i depends on how you develop the environment: do you use internal devs to provide and internal feature flagging, or pay for a SaaS. Usually the cost/income ratio is quite good.

## Where to start

We will use [ConfigCat](https://www.configcat.com) in the example, but any other SaaS provider will work very similar way. First of all, register on the webpage and set our first feature.

![_config.yml]({{ site.baseurl }}/images/feature-flag3.png)

To connect the API with our local code, follow the steps described on the website. As we use Python right now, install configcat-client.

```shell
pip install configcat-client
```

Use your favourite IDE, and test.

<iframe src="https://pastebin.com/embed_iframe/13A2DP52" style="border:none;width:100%;height:210px"></iframe>

If this works, the good old if-else statements do the rest of the magic, and marketing specialist, business owners click stuff on the webpage.

### Example opportunities

#### Iterative rollout, A/B testing

Start with like 5% of the customers, and if everything seems fine, increase again and again.

![_config.yml]({{ site.baseurl }}/images/feature-flag4.png)

#### Geolocation based appearance

## Customer story from a real-life example

### Domain

### The problem

### The solution

## Results and evaluation

## Providers on the market (SaaS)

- [ConfigCat](https://www.configcat.com) (we use this one)
- [Featureflags.io](https://featureflags.io) (Opensorce)
- [LaunchDarkly](https://launchdarkly.com)
- [Vwo](https://vwo.com)
- [Cloudbees](https://www.cloudbees.com)
- [Hackle](https://hackle.io/)
- [Split.io](https://www.split.io)

## Summary
