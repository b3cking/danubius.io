---
layout: post
author: Istvan Kis
title: Why is user input validation mandatory?
image: input-validation-1.jpg
keywords: input validation, attack vectors, server side validation, client side validation, web, hacker, secure development
---
{% include image.html src="/images/input-validation-1.jpg" alt="" %}

In this blog post we will cover the most important input validation techniques on the web, some attacks againts them, and highlight the importance of sanitizing data sent by the end user.

<!--more-->

## Purpose of input validation

On the internet many endpoints are publicly available, therefore not only our targeted user group will be able to use our service, but malicious hackers as well. Their goals may vary, but one of the most desired one is to get a remote shell to the server computer, and later collect different kind of information, use cpu to mine cryptocurrency, etc. there. As a developer we need to build secure application, and sanitizing input data is only one, but very important part of this process.

## Difference between client and server side validation

{% include image.html src="/images/input-validation-2.png" alt="Client and server validation draw" %}

As you can see in the figure above, the basic scheme is quite easy: client side validation runs on the user's computer, while server side validation happens on the server side. There are many advantages and disadvantages both of them, and we not plan to summarize all of them here, only a couple.

**Client side** validation can be fast, does not affect our serving infrastructure, the performance is based only on the client computer's resources, but it runs on users' computer, so it is very easy to bypass. It is written usually in scripting language (most often Javascript) so the browser can run it.

**Server side** validation is slow, every check needs at least 1 http request-response pair until it gets results and/or feedback to the user, our servers' resources are used, but in exchange it can be far more secure. Any server-side language can be used, usually .NET, PHP, Python, Java and related frameworks are the chosen ones.

Now that we understand the basics (if you need more info, [check here](https://www.youtube.com/results?search_query=client+and+server+side+validation)), let's move on to specific implementation of these..

## Validation techniques

Let's iterate through some common validation methods, which are used on the web frequently. String (text) validation will be completely skipped (if you interested in it, check [this OWASP cheat sheet](https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html)), and jump instantly to file uploads. Note that the mentioned techniques are not perfectly secure by definition, we need to understand where and how they can be mixed to mitigate the risks.

### File extension

The easiest way is to check the extension of the uploading file. Easy to implement, easy to bypass. On Windows systems, the file extension is still something, it helps define file type, while Unix based operating systems use other ways (eg. magic numbers - see it later) to 

### File type

### Magic number

### File name

### File length

### File content

## Client side attacks

Javascript bypass with Developer Tools or Burp

Turn off JS

Mime type (eg. image/jpeg)


## Server side attacks

Extension (.phpX)

Magic numbers

