---
title: OAuth Security and Testing Guide - Authorization Server
author: Paolo Serra
date: 2021-08-03 10:00:00
categories: [Topic, OAuth2]
toc: true
---




# Session Hijacking

| Vulnerable Parameters        | Requirements     |
|:-----------------------------|-----------------:|
| redirect uri          | Leaky redirect uri page     |


## Attack

Summary
: The attacker could hijack the authorization code issued for a "leaky" ***redirect_uri***, then apply the leaked code on real Client's callback to log in Victim's account ([example](http://homakov.blogspot.com/2014/02/how-i-hacked-github-again.html)).


Let's imagine: you have been found a redirect URI page vulnerable (i.e Path traversal, XSS, Open Redirect and so on), but you don't know actually what you can do because the vulnerability is pretty useless (most of the time XSS are not useless, but they can be). At this point, how can you increase the impact?

As you start the OAuth process, the first request is the authorization request, where the redirect_uri is




## Defense
During the client authentication (when the client exchange the code for the token - token request), it's important to make sure to insert the ***redirect_uri*** parameter in the token request in order to tie the redirection to the session.
From the "OAuth2 In Action" book:
>According to the OAuth specification, if the redirect URI is specified in the authorization request, the same URI must also be included in the token request.


## Bypass
Sed ipsum felis, fringilla ut mattis non, sodales nec lectus. Morbi volutpat tortor at leo euismod, id lobortis ligula sodales.
