---
layout: post
title: Basic web security mechanisms
subtitle: Explanation of CSP, SOP and CORS
categories: Web
cover-img: /assets/img/web/cors_error.png
thumbnail-img: /assets/img/web/chrome-security.png
tags: [CSP, SOP, CORS]
comments: true
---
# Intro :footprints:

All the following web security mechanisms you'll see came as solution to an exploit / attack vector. As a result, in my explanation you can see the exploit that the mechanism stop (apparently).

# Content Security Policy (CSP) :oncoming_police_car:

Website owners want to know that his website’s content is under control. It's include:

- images come from its own domain
- the scripts are loaded only from www.safescript.com domain
- and even ensure that all of its content is loaded using TLS.

In general CSP lets the website owner **control what resources the user agent is allowed to load for what page.**
To implement this feature you need to place HTTP header called: Content-Security-Policy and has the following syntax: **Content-Security-Policy: <policy name>**
There are many policies to this HTTP header that contain a string with a policy directive:

- <u>default-src ‘self’</u> - serves as a fallback/backup for the other CSP fetch directive (control locations from which certain resource types - image, fonts and scrtipts may be loaded) and it comes with self to mention - "I allow to load only resource from my own (self) domain".
- <u>script-src <source></u> - specifies valid sources for JavaScript.
  As I said before when all the policies directive mentions the **self** keyword it refers to the origin from which the protected document is being served, including the same URL scheme and port number.

# Same Origin Policy (SOP) :tea:

Domain A can access resources from domain B → problem. why? Let’s imagine that domain A is an attacker website and domain B is your bank website. Now, if the attacker found XSS vulnerability (I'll explain about this in another article) he can load his own malicious script from domain B.
**Same Origin Policy is a critical security mechanism that restricts from origin A to load resources from origin B. Two URLs have the same origin if the <u>protocol</u>, <u>port</u> and <u>host</u> are the same for both.**

# Cross-Origin Resource Sharing (CORS)

When website A wants to get resources (files, images, videos, etc...) from website B it may cause security issues because maybe the other side - A for example is an attacker that uses my website resources and uses them against me: Phishing, CSRF, etc…
For security reasons, browsers restrict HTTP cross origin requests initiated from script: JS code served from https://domain-a.com uses XMLHttpRequest to make requests to https://domain-b.com/data.json. As a result XMLHttpRequest and FetchAPI follow the **Same Origin Policy** (we already talked about), this means that **domain-a can access domain-b’s data if and only if the right CORS headers like Allow-Control-Allow-Origin are associated with the request.**

![cors_usage_illustrate](/assets/img/web/cors.png)

So after we understand the process behind the HTTP request between the two sides:
Cross Origin Resource Sharing is an HTTP header based mechanism that allows a server to indicate any origins (domain, scheme or port) - other than its own, which a browser should permit loading of resources. Behind the scenes, the browser requests the server hosting the cross-origin resource, in order to check that the server will permit the actual request:

![image-20230203032619514](/assets/img/web/cors_header_wildcard.png)

(The asterisk means 'any origin')

## CORS Default Permissions :key:

Let's summarize what CORS allows us or not to do:

### Import - Enabled

Import requests that are enabled by SOP, so one site can insert our site into an iframe, or use our javascript by script tag and the ‘src’ attribute.

### Write - Enabled

Writing requests are enabled by SOP, write operations refer to clicking on links, performing redirects, approving forms and receiving data through the address bar.

### Read - Not enabled

Readings from another site are prohibited by SOP, so site one can’t steal the content from site two.

# Can I pass this shit ?

## JSONP

"JSON with padding". This method enables bypassing Same Origin Policy because it enables transfering JSON data between different origins within a script tag:

![image-20230126230629233](/assets/img/web/jsonp_script_tag_implementation.png)

[This program will print all the json data from the different origin to the browser console]
Notice the callback parameter, as you already saw it’s called JSONP - JSON with Padding and the padding means the function (the function that is mentioned in the callback parameter) that warps the JSON payload. If you don’t specify a callback parameter so it won’t be consider as JSONP and probably disabled by the browser because X-Content-Type-Options header or the CORB - Cross Origin Read Blocking (Chrome security feature that came to prevent delivering a Cross Origin requests to a webpage).

