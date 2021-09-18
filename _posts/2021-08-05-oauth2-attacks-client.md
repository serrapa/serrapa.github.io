---
title: OAuth Security and Testing Guide - Client
author: Paolo Serra
date: 2021-08-03 10:00:00
categories: [Topic, OAuth2]
toc: true
---

# Intro

The OAuth ecosystem is complex. As a developer, you should follow the OAuth core specification as best as you can. Additionally, some tutorials from the OAuth community may be helpful to get into the topic.  If you are particularly keen on security, there is the "OAuth 2.0 Threat Model and Security Considerations specification" ([RFC 6819](https://datatracker.ietf.org/doc/html/rfc6819)). That is a useful resource to improve your knowledge and understanding of how you can properly carry on attacks or defend OAuth entities.



---


# Account Takeover

The typical scenario for this attack requires that the target website allows logging in either via email registration or via OAuth through multiple providers. It is even possible to face another scenario where the end-user can link his account to a third-party account in his profile page (i.e. Facebook, Linkedin, Google and so on) once he's already signed up.


*1° Scenario*
: Register a new account with the victim's email on the target website. From this time, you own the credentials but still not the account takeover, because an email verification will be probably required. When the victim registers himself/herself by using OAuth (with Facebook or Google, for example), this authorization will bypass the verification required and you will have another way to log in with the victim's account.  

| Functional Requirements             |
|:------------------------------------|
| Multiple Authentication Methods     |


*2° Scenario*
: First of all, this scenario depends on how the website manages the linking process. It might use IDs parameters, xxx or yyy. What I can suggest, it's to see the workflow and try to find out if the process is flawed. For example, if IDs are used to tie the third-party profile to the account, try changing them with the victim's id, thus the victim will have his account tied to the attacker's account.

| Functional Requirements             |
|:------------------------------------|
| Link a third-party Account in the profile page        |



*3° Scenario*
: Register a new account on the Resource Server with the victim’s email (even if email verification is required), then log in to the target website via OAuth with the account just created (with the victim’s email) and see what the target website does. Obviously, the victim must have already a registered account on the website and signed up via email. If you’re lucky, it logs you in the victim's account.  

| Functional Requirements             |
|:------------------------------------|
| The victim signed up by email       |
| The victim didn't sign up by OAuth  |

## Defence
**1° and 2°Scenario**: Require that the email verification is completed before logging in to the user. Otherwise, either don't let the user enter with OAuth when there's already another account created with the same email or let the user enter, but let him know someone else has already created an account and if it was him or not then ask him to change the password.

**3° Scenario**: This problem should not be fixed client-side but server-side. In fact, the RS should not allow using OAuth with an account not verified. But we trust no one, so to implement remediation, follow the previous one.

#### Reference
- 1° Scenario: [https://hackerone.com/reports/1074047](https://hackerone.com/reports/1074047)
- 2° Scenario: [https://0xgaurang.medium.com/case-study-oauth-misconfiguration-leads-to-account-takeover-d3621fe8308b](https://0xgaurang.medium.com/case-study-oauth-misconfiguration-leads-to-account-takeover-d3621fe8308b)

---


# CSRF
Sed ipsum felis, fringilla ut mattis non, sodales nec lectus. Morbi volutpat tortor at leo euismod, id lobortis ligula sodales.


## Defense
test

---

# OAuth Misconfigurations
Sed ipsum felis, fringilla ut mattis non, sodales nec lectus. Morbi volutpat tortor at leo euismod, id lobortis ligula sodales.

## Client Credentials
Sed ipsum felis, fringilla ut mattis non, sodales nec lectus. Morbi volutpat tortor at leo euismod, id lobortis ligula sodales.

### Defence
test

## Redirect URI
Sed ipsum felis, fringilla ut mattis non, sodales nec lectus. Morbi volutpat tortor at leo euismod, id lobortis ligula sodales.


### Defence
test

## OAuth Flow Misconfigurations
Sed ipsum felis, fringilla ut mattis non, sodales nec lectus. Morbi volutpat tortor at leo euismod, id lobortis ligula sodales.


### Flow Change
Sed ipsum felis, fringilla ut mattis non, sodales nec lectus. Morbi volutpat tortor at leo euismod, id lobortis ligula sodales.


### Defence
test

### Improper Impletation Implicit Flow
Sed ipsum felis, fringilla ut mattis non, sodales nec lectus. Morbi volutpat tortor at leo euismod, id lobortis ligula sodales.


### Defence
test

---
# Token Attacks

## Token Stealing Vectors
Sed ipsum felis, fringilla ut mattis non, sodales nec lectus. Morbi volutpat tortor at leo euismod, id lobortis ligula sodales.


### Defence
test

---

## Token Hijacking

| Limitation                          |
|:------------------------------------|
| Only tested for Facebook              |
| Need to have access to the token    |

From Facebook Documentation:
> To understand how this happens, imagine a native iOS app that wants to make API calls, but instead of doing it directly, communicates with a server owned by the same app and passes that server a token generated using the iOS SDK. The server would then use the token to make API calls. The endpoint that the server uses to receive the token could be compromised and others could pass access tokens for completely different apps to it. This would be obviously insecure

It's only possible to verify this vulnerability if you can control and have access to the ***access_token***. Check if the client is validating the token issued by the AS (precisely the Token endpoint), examining if it was issued for ONLY the client itself. If not, an attacker can use a victim’s access token issued for other clients to log in as the victim himself.   To do so, replace the access_token with another one issued by another application (client).

### Defence
Access tokens should never be assumed to be from the app that is using them. The client must check which client the token has been issued for.

#### Reference
- [https://hackerone.com/reports/314808](https://hackerone.com/reports/314808)

---

## Token Leakage

If you can check in the client side, the token should never be sent via GET method  .

### Defence
Send the token in the request body

---

## Lack Of Refresh Token
Sed ipsum felis, fringilla ut mattis non, sodales nec lectus. Morbi volutpat tortor at leo euismod, id lobortis ligula sodales.


### Defence
test
