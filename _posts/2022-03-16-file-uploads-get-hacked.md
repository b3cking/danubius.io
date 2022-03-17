---
layout: post
author: Istvan Kis
title: How your file uploads may get hacked?
image: input-validation-1.jpg
keywords: input validation, attack vectors, server side validation, client side validation, web, hacker, secure development
---
{% include image.html src="/images/input-validation-1.jpg" alt="" %}

In this blog post we will cover the most important file input validation techniques on the web, some attacks againts them, and highlight the importance of sanitizing data sent by the end user with a case study.

<!--more-->

## Purpose of input validation

On the internet many endpoints are publicly available, therefore not only our targeted customer group will be able to use our service, but malicious hackers as well. Their goals may vary, but the most desired one is usually to get a remote shell to the server computer, and later collect different kinds of information, use CPU to mine cryptocurrency, etc. there. As a developer we need to build secure applications, and sanitizing input data is only one, but very important part of this process.

## Difference between client and server side validation

{% include image.html src="/images/input-validation-2.png" alt="Client and server validation draw" %}

As you can see in the figure above, the basic scheme is quite easy: client side validation runs on the user's computer, while server/application side validation happens on the server side. There are many advantages and disadvantages of both of them, and we don't plan to summarize all of them here, only a couple.

**Client side** validation can be fast, does not affect our servers/infrastructure, the performance is based only on the client computer's resources, but it is very easy to bypass, because it runs on local machine. It is written usually in scripting language (most often Javascript or Typescript) so the browser can run it by itself.

**Server side** validation is slower, every check needs at least 1 http request-response pair until it gets results and/or feedback to the user. Our servers' resources are used, but in exchange it can be far more secure. Any server-side language can be chosen, usually .NET, PHP, Python, Java and related frameworks are the most famous ones.

Now that we understand the basics (if you need more info, [check here](https://www.youtube.com/results?search_query=client+and+server+side+validation)), let's move on to specific implementation of these.

## File validation techniques

Let's iterate through some common validation methods, which are used on the web frequently. String (text) validation will be completely skipped (if you're interested in it, check [this OWASP cheat sheet](https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html)), and jump instantly to file uploads. Note that the mentioned techniques are not perfectly secure by definition, we need to understand where and how they can be mixed to mitigate the risks.

### File extension filtering

```javascript
if(file.extension != ".jpg"){
    alert("Not valid file extension.");
}
```

The easiest way is to check the extension of the uploading file. On Windows systems, the file extension still means something, it helps to define file type, while Unix based operating systems rely on other methods (eg. magic numbers - see it later) to get information about files. This approach is easy to implement, but also easy to bypass (changeable locally), depending on the ``file.extension`` calucation method, blacklisting or whitelisting file extensions, etc.

### File type filtering

Different techniques are available, let's see the two most popular ones.

#### Mime type

When a client uploads a file via http/s, the header always contain a "``Content-Type:`` " part, where the value is in type/subtype format, e.g. image/jpeg for a jpg picture. It is almost the same as file extension filtering, because mime types are based on extension of the file.

#### Magic number

Magic numbers are the first few bytes of a file, which is completely unique to a certain file type. Linux ``file`` command is based on this string.

{% include image.html src="/images/input-validation-3.png" alt="File information on OS/X" %}

Jpegs are usually allowed in a picture uploader web form, for testing purpose let's assume we are an attacker, and want to upload a malicious binary or shell script over a magic number validation.

{% include image.html src="/images/input-validation-4.png" alt="PHP script magic number" %}

We can see, that hex signature of bash script is ``23 21``, which is [accurate](https://en.wikipedia.org/wiki/List_of_file_signatures). We want to change this beginning to ``FF D8 FF DB``, because that's the signature of jpegs, so we add 4 chars to the beginning of the file, e.g. AAAA. These will be the bytes we want to change.

{% include image.html src="/images/input-validation-5.png" alt="Extra chars for magic numbers" %}

Let's change and check it with ``file``.

{% include image.html src="/images/input-validation-6.png" alt="Bypass magic numbers" %}

This bash script won't be able to run without change, but imagine the same with a PHP before the starting of ``<?php``, a malicious backend code can be run after bypassing magic number validation. (Upload as ``.php``, but with jpeg magic number.)

### File name, length, content filtering

Filtering control characters (slash, backslash, asterisk, semicolon, etc.) is mandatory, lengths check is pretty straight-forward, and usually not a real problem for attacker (images are bigger than a functional reverse shell). Fully functional content filtering is complicated, usually not implemented in a single webapp, only the previously mentioned content typing (extension, etc.) are used.

## Client side attacks

These validations are unsafe, and we do not use these to provide security, but to give customers fast feedbacks and pleasure. How can we bypass?

- First and foremost you can completely switch off Javascript in your browser.
- Use browser Developer Tools for changing Javascript methods, even if those are obfuscated, use de-obfuscator.
- Install a middle-proxy between your client and the server, such as Burp, and manipulate every requests after client validation.
- Add or change content type there.

## Server side attacks

- Extension validation can be incomplete (e.g. filtering all .phpX stuff, but forgetting .phtmlX or similar, which is still processed by web server and php plugin)
- Content checks and filtering.
- Magic numbers we already covered.

There are many other validation techniques, which were not mentioned here, know and use them properly for building a secure applications. Remember that security bugs may not be similar to functional ones, where you can fix errors even after customers faced it. In certain situations you may not have the chance to do this after critical, escalated unauthorised access over a security hole.

Let's see how an insufficient validation may cause deeper problems.

## A short case study

We use a vulnerable demo box from [Tryhackme](https://tryhackme.com/).

{% include image.html src="/images/input-validation-8.png" alt="Frontpage layout" %}

Let's see what's on client side in html/js code.

{% include image.html src="/images/input-validation-7.png" alt="Javascript validation" %}

We can see that there is a length, a jpeg (magic number) and an extension validator. We need to bypass every one of them somehow. With Wappalyzer, we can see that the backend is served by NodeJS, which is important to know, because of the type of possibly usable reverse shells.

{% include image.html src="/images/input-validation-9.png" alt="NodeJS at backend" %}

When we try to upload a non-jpeg file, an error occurs, so the javascript does its job *well*. So right now we are able to upload jpegs, but we don't know where those are stored. Let's try to investigate web directory structure with Gobuster.

{% include image.html src="/images/input-validation-10.png" alt="Interesting directories" %}

Many of them are http error 301, because directory listing is prohibited, but ``assets`` and ``content`` can be good candidates for storing the uploaded files in. The THM box helps us with a keyword list, which contains 3-letter words. Maybe those are directories, files or else, but it's still a good hint. I only show the final solution below, because the investigation went on different failed ways. Content dir contains uploaded stuff.

{% include image.html src="/images/input-validation-11.png" alt="File list inside content directory" %}

After uploading a file, it appears here with a randomly chosen 3-char long file name and ``jpg`` extension. From previous directory listing we realised that an ``admin`` directory is also reachable without any error code (200). This is how it looks like.

{% include image.html src="/images/input-validation-12.png" alt="Admin runner" %}

It seems that the ``module`` directory contains scripts which can be executed by this form. The problem is that our uploads are not in that dir, and we can only upload jpegs. Earlier we acknowledged that NodeJS is serving the backend, let's use [this payload](https://github.com/appsecco/vulnerable-apps/tree/master/node-reverse-shell) to get a reverse shell, and try to upload it as jpeg. Make sure that IP and port is correct, that's where you should listen for reverse connection.

{% include image.html src="/images/input-validation-13.png" alt="Reverse shell" %}

Let's upload it and check the files again in ``content`` directory with gobuster, so we know the actual name of it.

{% include image.html src="/images/input-validation-14.png" alt="Shell as jpeg" %}

We can download the ``LGV.jpg`` from ``content`` directory to make sure that it's our file. It is. Let's go back to the admin page and examine that form, capture the request with Burp proxy.

{% include image.html src="/images/input-validation-15.png" alt="Command call from admin interface" %}

We see that the param is a direct POST parameter. Now start a ``netcat`` listener on the previously configured IP and port (10.9.125.225:1234 in my case). If it runs, we should call that "jpg" from the modules directory with relative path. We are pretty sure that the OS is a Linux or other NIX-based system, because it answers ICMP pings and nmap base scans. (Microsoft firewall drops every ICMP request by default.)

{% include image.html src="/images/input-validation-16.png" alt="Root access to the server" %}

The connection was successful, we have **root shell** to the server now, and we didn't even need to handle the jpeg situation, backend was able to call a jpg directly. *Good* enough. The room's input validation was way below average, and the Nginx/NodeJS shouldn't be run as root, but it still shows us the importance of input validation.

## Lessons learned

Sanitize every single user input, and handle those with proper care on both client and server side. Do not assume anything, check and test those values!
