---
title: OAuth Security and Testing Guide - Client
author: Paolo Serra
date: 9999-08-03 10:00:00
categories: [Topic, OAuth2]
toc: true
author: paoloserra
media_subpath: /images/
image:
  src: wallpapers/jujutsukaisen.jpeg
---


> Change the date of this post to make it visible.
{: .prompt-danger }

## Intro

The OAuth ecosystem is complex. As a developer, you should follow the OAuth core specification as best as you can. Additionally, some tutorials from the OAuth community may be helpful to get into the topic.  If you are particularly keen on security, there is the "OAuth 2.0 Threat Model and Security Considerations specification" ([RFC 6819](https://datatracker.ietf.org/doc/html/rfc6819)). That is a useful resource to improve your knowledge and understanding of how you can properly carry on attacks or defend OAuth entities.

The following paragraphs describe the theory behind of the most attacks and vulnerabilities about OAuth scenarios, so no pratical and technical details will be covered. 

Being said that, let's dive into the attacks against OAuth Client

---


## Account Takeover

The typical scenario for this attack requires that the target website allows users to log in either via email registration or via OAuth through multiple providers. It is even possible to face another scenario where the end-user can link his account to a third-party account in his profile page (i.e. Facebook, Linkedin, Google and so on) once he's already signed up.


*1° Scenario*
: Register a new account with the victim's email on the target website. From this time, you own the credentials but still not the account takeover, because an email verification will be probably required. When the victim registers himself by using OAuth (with Facebook or Google, for example), this authorization will bypass the verification required and you will have another way to log in with the victim's account.

| Functional Requirements         |
| :------------------------------ |
| Multiple Authentication Methods |


*2° Scenario*
: First of all, this scenario depends on how the website manages the linking process. It might use IDs parameters, xxx or yyy. What I can suggest, it's seeing the workflow and trying to find out if the process is flawed. For example, if IDs are used to tie the third-party profile to the account, try changing them with the victim's id, thus the victim will have his account tied to the attacker's account.

| Functional Requirements                        |
| :--------------------------------------------- |
| Link a third-party Account in the profile page |



*3° Scenario*
: Register a new account on the OAuth Provider with the victim’s email (even if email verification is required), then log in to the target website via OAuth with the account just created (with the victim’s email) and see what the target website does. Obviously, the victim must have already a registered account on the website and signed up via email. If you’re lucky, it logs you in the victim's account.

| Functional Requirements                                        |
| :------------------------------------------------------------- |
| The victim signed up by email                                  |
| The victim doesn't have an account on the Authorization Server |

### Defence

**1° and 2°Scenario**: Require that the email verification is completed before logging in to the user. Otherwise, either don't let the user enter with OAuth when there's already another account created with the same email or let the user enter, but let him know someone else has already created an account and if it was him or not then ask him to change the password.

**3° Scenario**: This problem should not be fixed client-side but server-side. In fact, the RS should not allow using OAuth with an account not verified. But we trust no one, so to implement remediation, follow the previous one.

#### Reference

- 1° Scenario: [https://hackerone.com/reports/1074047](https://hackerone.com/reports/1074047)
- 2° Scenario: [https://0xgaurang.medium.com/case-study-oauth-misconfiguration-leads-to-account-takeover-d3621fe8308b](https://0xgaurang.medium.com/case-study-oauth-misconfiguration-leads-to-account-takeover-d3621fe8308b)
- 3° Scenario: [https://portswigger.net/web-security/oauth#unverified-user-registration](https://portswigger.net/web-security/oauth#unverified-user-registration)

---

## Cross Site Request Forgery (CSRF)

The *state* parameter is optional in OAuth, but if you care about security, it becomes mandatory. When we talk about CSRF in OAuth, there are two different impacts:

 *1°* Scenario*
 : **Victim logs in Attacker's Account**: if *state* parameter is missing or not validated, an attacker can link his own profile to the victim's session. In this way, under some circumstances (not always), the victim cannot realize he is in another profile and may insert sensitive information in it. 

*2° Scenario*
: **Attacker logs in Victim's Account** If *state* parameter is missing or not validated and the attacker has access to the victim's *code* that has not been used yet, he can use it to request a token and access to the victim's resources.

### Defense

Both scenarios required the validation of the *state* . This mitigation must be exactly done as for a typical Anti-CSRF mechanism, so tying the *state* to the user's session.

#### Reference

[https://portswigger.net/web-security/oauth#flawed-csrf-protection](https://portswigger.net/web-security/oauth#flawed-csrf-protection)

---

## Client Credentials

Most related to mobile applications where developers hardcode secrets inside the code, but even if you get access to the Client's machine, the secrets to communicate with the Authorization Server become yours and you can request tokens in behalf of the Client by that moment.  

### Defence

Client's secret must be stored securely because are very important. Mobile applications are a separate discussion because require a more elaborate "mitigation" in order to avoid exposure.
 

#### Reference

>to be add

---

## Redirect URI

Registering the URI callback is a ***critical*** step, because a bad registration could lead to serious impacts. 
Even here there are different scenarios to consider:

*1°* Scenario*
 : *Stealing*  ***Code*** *through the Referer* : 

*2°* Scenario*
 : *Stealing*  ***Token*** *through Open Redirect* : 


### Defence

>to be done

#### Reference

[https://portswigger.net/web-security/oauth#leaking-authorization-codes-and-access-tokens](https://portswigger.net/web-security/oauth#leaking-authorization-codes-and-access-tokens)

---


## Flow Change

>to be done

### Defence

Force the use of the response_code wanted

#### Reference

>to be add

---

## Improper Impletation Implicit Flow

Let's start with the fact that the Implicit flow should be mainly used for single-page applications, but it is also often used in classic client-server web applications.  
In this flow, the access token is sent from the OAuth service to the client application via the user's browser as a URL fragment. The client application then accesses the token using JavaScript. Here we need to consider what the application does:

- if the application wants to maintain the session after the user closes the page, it needs to store the current user data (normally a user ID and the access token) somewhere and that could lead to a security problem. Typically, the client application will often submit this data to the server in a POST request and then assign the user a session cookie. In this scenario, the server does not have any secrets or passwords to compare with the submitted data, which means that it is implicitly trusted. As a result, this behaviour can lead to a serious vulnerability if the client application doesn’t properly check that the access token matches the other data in the request. At this point, an attacker can simply change the parameters sent to the server to impersonate any user.


### Defence
Check that the access token matches the other data in the request and tie it to the user’s data and the session cookie.

#### Reference

[Academy Portswigger - OAuth](https://portswigger.net/web-security/oauth#improper-implementation-of-the-implicit-grant-type)
 
---

## Token Stealing Vectors

>to be done
-  Open Redirect
- Javascript that handles query parameters and URL fragments 
- XSS
- HTML Injection
- Path Traversal + other vulnerabilities on another page

### Defence

>to be done

#### Reference

>to be add
 
---

## Token Hijacking

| Limitation                                                     |
| :------------------------------------------------------------- |
| Only tested for Facebook                                       |
| Need to have access to the token                               |
| Need to have an external app that uses the same OAuth Provider |

From Facebook Documentation:
> To understand how this happens, imagine a native iOS app that wants to make API calls, but instead of doing it directly, communicates with a server owned by the same app and passes that server a token generated using the iOS SDK. The server would then use the token to make API calls. The endpoint that the server uses to receive the token could be compromised and others could pass access tokens for completely different apps to it. This would be obviously insecure

It's only possible to verify this vulnerability if you can control and have access to the ***access_token***. Check if the client is validating the token issued by the AS (precisely the Token endpoint), examining if it was issued for ONLY the client itself. If not, an attacker can use a victim’s access token issued for other clients to log in as the victim himself. To do so, replace the access_token with another one issued by another application (client).

### Defence

Access tokens should never be assumed to be from the app that is using them. The client must check which client the token has been issued for.

#### Reference

- [https://hackerone.com/reports/314808](https://hackerone.com/reports/314808)

---

## Token Leakage 

Tokens and codes are prone to leakage and ensuring that's not happen is not as obvious as it could be. Someting it may be intentionally or unintentionally:

- as the attacker you cannot see what the Client does server side, but somehow if you find out the token is sent to the Resource Provider via a GET request, there is a Leakage since the param is certainly logged somewhere where it shouldn't be. 

- they may be leaked via *Referer* header when external images, scripts, or CSS content is loaded. 

- It can be included in the dynamically generated JavaScript files that are executed from external domains via *<script>* tags

### Defence
Always send sensitive information in the ***body*** of a request and don't include them in the Referer header. It is also important to not include them in the dynamically generated JavaScript files.

#### Reference

[https://portswigger.net/web-security/oauth/preventing](https://portswigger.net/web-security/oauth/preventing)

---

## Lack Of Refresh Token

>to be done

### Defence

I don't know who, how and why, but one day someone got up and came up with an idea: refresh tokens.

#### Reference

>to be add
