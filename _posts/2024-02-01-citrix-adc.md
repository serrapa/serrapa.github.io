---
title: Citrix ADC - Unexpected treasure
author: Paolo Serra
date: 2024-02-01 10:00:00
categories: [Web Security]
toc: true
author: paoloserra
media_subpath: /images/citrix-adc/
image:
  path: wallpaper.webp
  width: 100%
  height: 315
---

> **TL. DR**: *Setting secure rules for the RelayState parameter is a MUST when configuring **Citrix Application Delivery Controller (ADC)** and **Citrix Gateway as SAML Service Provider**, because an attacker can exploit a chain of three low-risk level vulnerabilities to compromise victims' accounts. By luring users to a malicious domain, attackers can steal session cookies and gain unauthorized access to the protected applications. The vulnerabilities chain poses a significant risk, potentially leading to Privilege Escalation or Account Takeover on organization's users.*


***This blogpost was firstly published by Yarix at this [link](https://labs.yarix.com/2024/03/citrix-adc-unexpected-treasure/)***.

## Introduction
As a Red Team, we are engaged in penetration testing and security assessments of applications, systems, and network infrastructures to test their security, bearing in mind the client business requirements and the specific context in which the target’s technologies are designed and deployed. A recent case involved assessing a **Single Sign-On** system designed to ensure controlled access to internal and external corporate portals. While SSO systems are common in corporate environments, the uniqueness of this activity stemmed from the contextualized scenario and technologies used and deployed.

During these types of assessments, it’s crucial to consider the SSO system independently. The individual corporate portals can be viewed as third-party entities that need to communicate with others (i.e. for the authentication and authorization phases), whereas the Single Sign-On system works autonomously. Often, it relies on well-established and tested technologies and tools, likely immune to common injection attacks and vulnerabilities. Engineers configuring these SSO typically focus on setting up the underlying technologies to ensure proper communications and user redirection.

This post will go through all the attack chain’s details discovered and linked to [CVE-2020-8300](https://nvd.nist.gov/vuln/detail/CVE-2020-8300), as confirmed by the Citrix HackerOne team. Although it was [patched](https://support.citrix.com/article/CTX297155/citrix-application-delivery-controller-citrix-gateway-and-citrix-sdwan-wanop-edition-appliance-security-update) by Citrix, further **actions are required** by customers for all versions at the time of writing. At the end of this article, we will present effective patches and demonstrate potential complications in case of misconfigurations.

## Scenario

A Single Sign-On system may be implemented to control access to corporate portals based on the organization's business units. SharePoint, Azure Active Directory and Citrix ADC can be used to create a workflow that enables individual employees to access a list of web portals. Each component plays a distinct role:

- SharePoint is responsible for listing links to corporate portals.
- Azure Active Directory (and thus Microsoft 365 Login) manages the authentication.
- Citrix ADC checks and validates proper rights to access to individual portals (authorization).

It's important to emphasize that no additional code, which may introduce unexpected vulnerabilities, needs to be written to develop this flow. Below is an illustrative representation of the workflow:

![Window shadow](user_workflow.png)
_User Workflow_

As indicated by the colored keys (blue and red), each user is granted access only to web portals relevant to their business unit of belonging. This aspect is crucial during testing, as it highlighted the potential for horizontal privilege escalation attacks. Additionally, certain web applications might have separate sets of application users — sometimes the domain account is only needed in order to land on the application, like a second authentication; otherwise, they rely on the same accounts as those in Azure Active Directory. In practical terms, compromising an account within the Single Sign-On (SSO) system could result in vertical privilege escalation within a specific web application.


#### A bit more technical (Citrix Overview)

As pentesters, our task of assessing the security of such a design requires us to follow a structured approach and have an open mindset. Our scenario was similar to that one just described. Thus, wondering how this flow works under the hood was the starting point. One of the first questions to address was: *“How does Citrix identify users and their permissions?”*

Citrix documentation[^footnote] explains how it can be configured as a Service Provider while using Azure AD as an Identity Provider, leveraging SAML as the authentication and authorization protocol. But what exactly is Citrix ADC?


As outlined in the documentation, Citrix ADC (aka Netscaler) is an application delivery controller that performs analysis, optimization, and protection of web application traffic. In a corporate environment, it may be configured as a Gateway[^fn-nth-2]:


> You can use NetScaler as a gateway at the perimeter of your organization’s internal network (or intranet) to provide a secure single point of access to the servers, applications, and other network resources that reside in the internal network.

For further technical details, when a user attempts to access an application protected by Citrix, the Service Provider (SP) evaluates the client’s request. If the client is not authenticated (meaning it does not have a valid *NSC_TMAA* or *NSC_TMAS* cookie), the SP redirects the request to the SAML Identity Provider (in our case, Azure AD with Microsoft 365). After that, the authentication proceeds through other two endpoints:

![Window shadow](citrix_endpoint.png)

The first endpoint is primarily responsible for verifying the SAML response. At this stage, the user is issued a session cookie and a code that must be validated in the second phase to complete the authentication process. The redirection to the second endpoint is based on the domain defined in the URL provided within the *RelayState* parameter.


#### Sequence Diagram of the Intended Authentication Flow

The following sequence diagram shows all the authentication steps starting from the user’s click on the SharePoint page (or simply navigating to the protected web application URL in the browser):

![Window shadow](scenario1.png)
_Authentication Sequence Diagram_

The last step (8) always returns a 200 OK HTTP response, but whether the user is presented with the web application page depends on the privilege he has. For example, the following page shows the HTML code returned if the user has no access privilege to the target:

![Window shadow](not_access.png)
_Access Denied_

## Flaw Analysis

During the testing, three different flaws were discovered even though they do not represent any high-security risks on their own. In this article, we assume what these vulnerabilities are and jump directly into their outcomes in terms of exploitability:

- A ***Cross-Site Request Forgery (CSRF)*** on the *login request* : this type of vulnerability on login appears particularly relevant in Single Sign-On contexts because it allows authentication with the attacker’s account. As a result, if the web app UI does not clearly indicate which account the victim is logged in with, victim users may enter sensitive information into that account. On the other side, how can an attacker exploit a situation where is able to make the victim log in with his own account? What would be the impact of such an attack? Mmh… It sounds challenging.

- ***Open Redirect Post-Based*** : this vulnerability is by design since it comes from the redirection to the URL set on the *RelayState* value (this behavior is intended according to Citrix). Furthermore, it is also very difficult to exploit a post-based open redirection for phishing attacks, thus the impact of this flaw can be lowered (considered as Info in my opinion).
- ***Host Header Injection*** : The code verification request grounds the evaluation on the Host header value even if there are no cookies sent. Taken as-is, this vulnerability is not exploitable. If it were reported, it would be considered as Low/Info in our opinion.

In general, any system suffering from these vulnerabilities should implement some specific countermeasures to mitigate the potential impact:

- *Cross-Site Request Forgery (CSRF)* : since the Identity Provider entrusts the login request, the vulnerability might not be fixed by Citrix. The only remediation would be to use an anti-CSRF token on the *RelayState* that must be validated during the request to `/cgi/selfauth` (authentication verification step). Moreover, the token must be tied to the session cookie and the *code* parameter. In doing so, an attacker would not be able to make the victim send a login request.
- *Open Redirect Post-Based* : this is an intendend behaviour, by-design, to redirect the user after the authentication is complete. The *RelayState* value is responsible for the redirection, so the only way to protect users is to enforce the use of secure rules: the url in the *RelayState* should start with `https://<target_domain>.com/` (the final slash is essential). The use of a rule is optional; therefore, the vulnerability is **by default**. However, the rule could still exist if it were bypassable, but this would be a configuration issue of the client using Netscaler.
- *Host Header Injection*: during the code validation, Citrix uses the *Host header* value. Moreover, since the *NSC_TMAS* cookie is also released at step 6 (see the diagram above) following the request to the `/cgi/samlauth` endpoint, it should be necessary to require the session cookie when sending the request to `/cgi/selfauth`, since it is not mandatory. In this way, the response should release and validate the *NSC_TMAS* session cookie only if the user sends the correct values of the *code* and the cookie value previously released.

Being said that, how should you protect your application? A Citrix article[^fn-nth-3] describes what you should do. But are you sure it's enough? Yes and no, because it describes how to mitigate the Open Redirect. Keep reading.


All these discovered vulnerabilities don’t pose significant security issues (CSRF certainly has a more relevant impact compared to the others); however, what could they lead to if chained together?


While the goal is to carry out an attack with high implications, it might be useful to start from the CSRF, a vulnerability that allows authenticating victims with an account under the attacker control. However, the exploitation would require a phishing phase and one (or more) prayers that the user enters sensitive information into an application where:

- The attacker has access privileges.
- The victim user does not notice that they are logged in with another account.


Given these considerations, the likelihood of the success for such an attack is rather low. So, why not shoot for something higher? For example, stealing the victim's session and gaining access to applications we're not supposed to?


## Attack Scenario

Thanks to the flaws discovered, we can try to reconstruct the entire flow by specifying the exact point at which each vulnerability can be exploited. First, let’s list the requirements for a successful attack:

- One valid account (regardless of the access privileges to the target app).
- Citrix ADC configured as a Service Provider.
- Missing rules on ***RelayState*** parameter or set in an insecure manner.

*Optional: You should have at least one valid account registered for the Identity Provider being used, regardless of their permissions on Citrix. That’s because it would be helpful to have a valid user to analyze the internal authentication flow of the target. Otherwise, the discovery phase of the attack and vulnerabilities would be complicated.*

The following sequence diagram shows all the illustrated steps that define the attack flow:

![Window shadow](scenario2.png)
_Attack Sequence Diagram_

As shown above, the diagram shows how an attacker can exploit the CSRF vulnerability to make the victim’s browser start a login request to Microsoft 365 (steps 5 to 7). However, to also exploit the Open Redirect the attacker must edit the URL in the *RelayState* parameter to point to his malicious domain (steps 7 and 8). In our case, the *RelayState* parameter was base64 encoded, with the URL defined after a null byte. The following is an example of a PoC:

```html
<html>
  <body>
    <form action="https://login.microsoftonline.com/<guid>/saml2" method="POST">
      <input type="hidden" name="RelayState" value="<target_domain>.<attacker_domain>" />
      <input type="hidden" name="SAMLRequest" value="<saml_request>" />
      <input type="submit" value="Submit request" />
    </form>
    <script>
      history.pushState('', '', '/');
      document.forms[0].submit();
    </script>
  </body>
</html>
```
{: file="Cross-Site Request Forgery (CSRF) PoC" }


Due to the code verification request, the *RelayState* should contain a URL starting with the target domain. Then, the malicious domain is appended to ensure successful verification in the subsequent step.


Upon obtaining the code value, the attacker must send and validate it on the `/cgi/selfauth` in order to get the victim’s session cookie (steps 10 and 11).

![Window shadow](selfauth.png){: .shadow .rounded-10 }
_Victim's session cookies in response_

The attack scenario demonstrated can be reproduced every time there are no rules set for the *RelayState* parameter. However, there might be some situations where it can still be exploited.


Upon setting *RelayState* properly, the attack chain is **broken**: the attacker can no longer obtain the *code* value, thus preventing them from completing the authentication flow to steal the victim’s session cookies. However, it's important to note:

- the ***code*** value is **sent via a GET request**, which means it is stored in cleartext somewhere (e.g. browser history, server logs, server proxy, ...).

- the authentication verification step (step 10) **does not require the session cookies** to be successful.

This allows an attacker, who has obtained the *code* value somehow, to complete the flow even though they do not have session cookies created and released to the victim’s browser during the pre-authentication request (step 8).


Furthermore, the ***RelayState*** expressions may be insecure and bypassable if not set properly. For example, the following expressions are bypassable (because of the missing slash) by using a malicious domain as `<target_domain>.<evil_domain>`: 
- `set authentication samlaction -relaystateRule 'AAA.LOGIN.RELAYSTATE.EQ("https://<target_domain>")'`
- `set authentication samlaction -relaystateRule 'AAA.LOGIN.RELAYSTATE.CONTAINS("https://<target_domain>")'`
 

### Impact

The attack chain enables an attacker to gain a valid session tied to a targeted victim’s account. This breach allows the attacker to elevate their privileges and access corporate web applications that would normally be off-limits, especially if their account is restricted to a limited set of applications. Furthermore, the attack can potentially grant additional privileges on individual web applications protected by Citrix, particularly if user identities are authenticated through the Identity Provider. In such scenarios, the victim’s role could have high-level privileges (e.g. admin), thereby allowing the attacker to significantly escalate their privileges within the affected web application (unauthorized access to sensitive corporate applications).


## Conclusion & Takeaways


The attack chain was reported to Citrix, which decided to confirm the vulnerability as Informative since they were already aware of the issues. Given the unclear decision not to implement more effective measures on their part (obviously highlighted in the report), the responsibility to prevent the attack is left to the Citrix panel administrators.

Below are the takeaways from this article:

- If secure rules are not configured on the *RelayState* parameter in the Citrix ADC panel, then the attack can be performed.

- Testing SSO scenarios with the purpose of breaking them always requires starting from the intended flow (don’t get caught up in injections or weird things while testing, they distract you from analysis). You need to know the flow like the back of your hand.

- Read the documentation or articles about the technologies under test. This helps you to develop a strong mindset behind the question: *“If I wanted to implement what the client did, how should I do it?”*

- Note down every issue or flaw discovered. Most of the time you won’t find XSS, SQLi, RCE or other high-impact injection vulnerability, as these technologies are well-tested against those kinds of attacks. **Arm yourself with smaller things to hit the logic**.

#### Hackerone Timeline

- 1<sup>st</sup> December 2023 – reported to the Citrix Bug Bounty Program.
- 4<sup>th</sup> December 2023 – triaged.
- 1<sup>st</sup> February 2024 – reviewed by Citrix staff.
- 3<sup>rd</sup> February 2024 – closed as Informative.
- 4<sup>th</sup> February 2024 – commented for discussion.
- 15<sup>th</sup> February 2024 – response for Citrix staff.

##### Links

[^footnote]: [https://docs.netscaler.com/en-us/citrix-adc/current-release/aaa-tm/authentication-methods/saml-authentication/citrix-adc-saml-sp.html](https://docs.netscaler.com/en-us/citrix-adc/current-release/aaa-tm/authentication-methods/saml-authentication/citrix-adc-saml-sp.html)

[^fn-nth-2]: [https://docs.netscaler.com/en-us/citrix-adc/current-release/getting-started-with-citrix-adc/where-citrix-adc-in-network.html](https://docs.netscaler.com/en-us/citrix-adc/current-release/getting-started-with-citrix-adc/where-citrix-adc-in-network.html#:~:text=Gateway%20%2D%20You%20can%20use%20NetScaler%20as%20a%20gateway)

[^fn-nth-3]: [https://support.citrix.com/article/CTX316577/citrix-application-delivery-controller-and-citrix-gateway-saml-configuration-reference-guide](https://support.citrix.com/article/CTX316577/citrix-application-delivery-controller-and-citrix-gateway-saml-configuration-reference-guide)
