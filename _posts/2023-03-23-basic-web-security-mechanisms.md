---
layout: post
title: Basic web security mechanisms
subtitle: Explanation of CSP, SOP and CORS
categories: Web
cover-img: /assets/img/web/cors_error.png
thumbnail-img: /assets/img/web/chrome-security.png
tags: [CSP,SOP, CORS]
comments: true
---
# Intro :footprints:

All the following web security mechanisms you'll see came as solution to an exploit / attack vector. As a result, in my explanation you can see the exploit that the mechanism stop (apparently).

# Content Security Policy (CSP) :oncoming_police_car:

Web administrator wants to know that his website’s content is under control, so the images come from its own domain or the scripts are loaded only from www.safescript.com domain and even ensure that all of its content is loaded using TLS. And in general CSP lets the web administrator **control what resources the user agent is allowed to load for what page.**
This comes with an HTTP header called: Content-Security-Policy and has the following syntax: **Content-Security-Policy: <policy name>**
There are many policies to this HTTP header that contain a string with a policy directive.
<u>default-src ‘self’</u> - serves as a fallback/backup for the other CSP fetch directive (control locations from which certain resource types may be loaded) and it comes with self to mention the origin from which the protected document is being served.
<u>script-src <source></u> - specifies valid sources for JavaScript.
When all the policies directive mentions the ‘**self**’ keyword it refers to the origin from which the protected document is being served, including the same URL scheme and port number.

# Same Origin Policy (SOP) :tea:

domain A can access resources from domain B → problem. why? Let’s imagine that domain A is an attacker website and domain B is your bank website. Now, do you want all your transactions, emails, signed papers and more to be visible for the attacker? no.
**Same Origin Policy is a critical security mechanism that restricts from origin A to load resources from origin B. Two URLs have the same origin if the <u>protocol</u>, <u>port</u> and <u>host</u> are the same for both.**

# Cross-Origin Resource Sharing (CORS)

When website A wants to get resources (files, images, videos, etc...) from website B it may cause security issues because maybe the other side - A for example is an attacker that uses my website resources and uses them against me: Phishing, CSRF, etc…
For security reasons, browsers restrict HTTP cross origin requests initiated from script: JS code served from https://domain-a.com uses XMLHttpRequest to make requests to https://domain-b.com/data.json. As a result XMLHttpRequest and FetchAPI follow the Same Origin Policy, this means that domain-a can access domain-b’s data if and only if the right CORS headers like Allow-Control-Allow-Origin are associated with the request.

[img of cors]

So after we understand the process behind the HTTP request between the two sides:
Cross Origin Resource Sharing is an HTTP header based mechanism that allows a server to indicate any origins (domain, scheme or port) - other than its own, which a browser should permit loading of resources. Behind the scenes, the browser requests the server hosting the cross-origin resource, in order to check that the server will permit the actual request.

## CORS Default Permissions :key:

### Import [enabled]

Import requests that are enabled by SOP, so one site can insert our site into an iframe, or use our javascript by script tag and the ‘src’ attribute.

### Write [enabled]

Writing requests are enabled by SOP, write operations refer to clicking on links, performing redirects, approving forms and receiving data through the address bar.

### Read [Not enabled]

Readings from another site are prohibited by SOP, so site one can’t steal the content from site two.

![image-20220723100654193](/assets/img/kerberos/KRB_AS_REQ.png)

## Keys :key:

### User secret key

A key that is derived from the user password hash.

### Session key

Used during the conversation between the TGS and the User. The user gets this key inside the KRB_AS_REP packet.

### Service (AP) key

This key derivative from the NTLM hash of service owner, (which can be a user or computer account) and used for the communication between the User and the Service.
