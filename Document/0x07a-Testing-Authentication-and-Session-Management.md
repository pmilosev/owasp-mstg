## Testing Authentication and Session Management on the Endpoint

The following chapter outlines authentication and session management requirements of the MASVS into technical test cases. Test cases listed in this chapter are focused on server side and therefore are not relying on a specific implementation on iOS or Android.  

### Verifying that Users Are Properly Authenticated

#### Overview

Applications often have different areas with, on the one hand public and non-privileged information and functions, and on the other hand sensitive and privileged information and functions. Users can legitimately access the first ones without any restriction; however, in order to make sure sensitive and privileged information and functions are protected and accessible only to legitimate users, proper authentication has to take place.  

#### Static Analysis

When source code is available, first locate all sections with sensitive and privileged information and functions: they are the ones that need to be protected. Prior to accessing any item, the application must make sure the user is really who he pretends to and that he is allowed to access the section. Look for keywords in the targeted programming language that are used to authenticate a user or to retrieve and check an existing session token (for instance: KeyStore, SharedPreferences, ...).

-- ToDo: Create more specific content about authentication frameworks, the framework need to be identified and if the best practices offered by the framework are used for authentication. This should not be implemented by the developers themselves.


#### Dynamic Analysis

The easiest way to check authentication is to try to browse the app and access privileged sections. When this cannot be done manually, an automated crawler can be used (for instance, try to start an Activity that contains sensitive information with Drozer without providing authentication elements; for further information, please refer to the official Drozer guide available at https://labs.mwrinfosecurity.com/tools/drozer/).

In case the app is exchanging information with a backend server, an interception proxy can be used to capture network traffic while being authenticated. Then, log out and try to replay requests while removing authentication information or not.
Further attacks methods can be found in the OWASP Testing Guide V4 (OTG-AUTHN-004)<sup>[1]</sup>.

#### Remediation

For every section that needs to be protected, implement a mechanism that checks the session token of the user:
- if there is no session token, the user may not have authenticated before;
- if a token exists, make sure this token is valid and that it grants the user with sufficient privileges to allow the user to access the section.

If any of these two conditions raise an issue, reject the request and do not allow the user to start the Activity.

#### References

##### OWASP Mobile Top 10 2016

* M4 - Insecure Authentication - https://www.owasp.org/index.php/Mobile_Top_10_2016-M4-Insecure_Authentication

##### OWASP MASVS

- 4.1: "If the app provides users with access to a remote service, an acceptable form of authentication such as username/password authentication is performed at the remote endpoint."

##### CWE

- CWE-287: Improper Authentication - https://cwe.mitre.org/data/definitions/287.html

##### Info

[1] OWASP Testing Guide V4 (OTG-AUTHN-004) - https://www.owasp.org/index.php/Testing_for_Bypassing_Authentication_Schema

##### Tools

* Drozer - https://labs.mwrinfosecurity.com/tools/drozer/

### Testing Session Management

#### Overview

All significant, if not privileged, actions must be done after a user is properly authenticated; the application will remember the user inside a "session". When improperly managed, sessions are subject to a variety of attacks where the session of a legitimate user may be abused, allowing the attacker to impersonate the user. As a consequence, data may be lost, confidentiality compromised or illegitimate actions performed.

Sessions must have a beginning and an end. It must be impossible for an attacker to forge a session token: instead, it must be ensured that a session can only be started by the system on the server side. Also, the duration of a session should be as short as possible, and the session must be properly terminated after a given amount of time or after the user has explicitly logged out. It must be impossible to reuse session tokens.

As such, the scope of this test is to validate that sessions are securely managed and cannot be compromised by an attacker.

#### Static Analysis

When source code is available, the tester should look for the place where sessions are initiated, stored, exchanged, verified and canceled. This must be done whenever any access to privileged information or action takes place. For those matters, automated tools or custom scripts (in any language like Python or Perl) can be used to look for relevant keywords in the target language. Also, team members knowledgeable on the application structure may be involved to cover all necessary entry points or fasten the process.

-- ToDo: Create more specific content about session management in frameworks, the framework need to be identified and if the best practices offered by the framework are used for session management. This should not be implemented by the developers themselves.


#### Dynamic Analysis

A best practice is first to crawl the application, either manually or with an automated tool, the goal being to check if all parts of the application leading to privileged information of actions are protected and a valid session token is required or not.

Then, the tester can use any intercepting proxy to capture network traffic between a client and the server and try to manipulate session tokens:
- modify a valid token to an illegitimate one (for instance, add 1 to the valid token or delete parts of the token);
- delete a valid token in the request to test if the targeted part of the application can still be accessed;
- try to log out and re-log in and check if the token has changed or not;
- when changing privilege level (step-up authentication), try to use the former one (hence with a lower authorization level) to access the privileged part of the application;
- try to re-use a token after logging out.

#### Remediation

In order to offer proper protection against the attacks mentioned earlier, session tokens must:
- always be created on the server side;
- not be predictable (use proper length and entropy);
- always be exchanged between the client and the server over secure connections (e.g. HTTPS);
- be stored securely on the client side;
- be verified when a user is trying to access privileged parts of an application: a token must be valid, correspond to the proper level of authorization;
- be renewed when a user is asked to log in again to perform an operation requiring higher privileges;
- be terminated when a user logs out or after a given amount of time.

It is strongly advised to use built-in session token generators as they are usually more secure than custom tokens. Such generators exist for most platforms and languages.

#### References

##### OWASP Mobile Top 10 2016

* M4 - Insecure Authentication - https://www.owasp.org/index.php/Mobile_Top_10_2016-M4-Insecure_Authentication

##### OWASP MASVS

* 4.2: "The remote endpoint uses randomly generated access tokens to authenticate client requests without sending the user's credentials."

##### CWE

- CWE-613 - Insufficient Session Expiration https://cwe.mitre.org/data/definitions/613.html

##### Info

[1] OWASP Session Management Cheat Sheet: https://www.owasp.org/index.php/Session_Management_Cheat_Sheet
[2] OWASP Testing Guide V4 (Testing for Session Management) - https://www.owasp.org/index.php/Testing_for_Session_Management

##### Tools

* Zed Attack Proxy
* Burp Suite

### Testing the Logout Functionality

#### Overview

Session termination is an important part of the session lifecycle. Reducing the lifetime of the session tokens to a minimum decreases the likelihood of a successful session hijacking attack.
 
The scope for this test case is to validate that the application has a logout functionality and it effectively terminates the session on client and server side.

##### Static Analysis 

If server side code is available, it should be reviewed that the session is being terminated as part of the logout functionality.
The check needed here will be different depending on the technology used. Here are different examples on how a session can be terminated in order to implement a proper logout on server side:
- Spring (Java) -  http://docs.spring.io/spring-security/site/docs/current/apidocs/org/springframework/security/web/authentication/logout/SecurityContextLogoutHandler.html
- Ruby on Rails -  http://guides.rubyonrails.org/security.html
- PHP - http://php.net/manual/en/function.session-destroy.php
- JSF - http://jsfcentral.com/listings/A20158?link
- ASP.Net - https://msdn.microsoft.com/en-us/library/ms524798(v=vs.90).aspx
- Amazon AWS - http://docs.aws.amazon.com/appstream/latest/developerguide/rest-api-session-terminate.html

#### Dynamic Analysis

For a dynamic analysis of the application an interception proxy should be used. The following steps can be applied to check if the logout is implemented properly.  
1.  Log into the application.
2.  Do a couple of operations that require authentication inside the application.
3.  Perform a logout operation.
4.  Resend one of the operations detailed in step 2 using an interception proxy. For example, with Burp Repeater. The purpose of this is to send to the server a request with the token that has been invalidated in step 3.
 
If the session is correctly terminated on the server side, either an error message or redirect to the login page will be sent back to the client. On the other hand, if you have the same response you had in step 2, then, this session is still valid and has not been correctly terminated on the server side.
A detailed explanation with more test cases, can also be found in the OWASP Web Testing Guide (OTG-SESS-006)<sup>[1]</sup>.

#### Remediation 

One of the most common errors done when implementing a logout functionality is simply not destroying the session object on server side. This leads to a state where the session is still alive even though the user logs out of the application. The session remains alive, and if an attacker get’s in possession of a valid session he can still use it and a user cannot even protect himself by logging out or if there are no session timeout controls in place.
 
To mitigate it, the logout function on the server side must invalidate the session identifier immediately after logging out to prevent it to be reused by an attacker that could have intercepted it.
 
Related to this, it must be checked that after calling an operation with an expired token, the application does not generate another valid token. This could lead to another authentication bypass.
 
Many mobile apps do not automatically logout a user, because of customer convenience. The user logs in once, afterwards a token is generated on server side and stored within the applications internal storage and used for authentication when the application starts instead of asking again for user credentials. If the token expires a refresh token might be used (OAuth2/JWT) to transparently reinitiate the session for the user. There should still be a logout function available within the application and this should work accordingly to best practices by also destroying the session on server side.

#### References

##### OWASP Mobile Top 10 2016
* M4 - Insecure Authentication - https://www.owasp.org/index.php/Mobile_Top_10_2016-M4-Insecure_Authentication

##### OWASP MASVS
-- TODO [Update reference "VX.Y" below for "Testing the Logout Functionality"] --
- 4.3: "The remote endpoint terminates the existing session when the user logs out."

##### CWE

-- TODO [Add relevant CWE for "Testing the Logout Functionality"] --
- CWE-312 - Cleartext Storage of Sensitive Information

##### Info

* [1] OTG-SESS-006 - https://www.owasp.org/index.php/Testing_for_logout_functionality
* [2] Session Management Cheat Sheet - https://www.owasp.org/index.php/Session_Management_Cheat_Sheet

### Testing the Password Policy

#### Overview

Password strength is a key concern when using passwords for authentication. Password policy defines requirements that end users should adhere to. Password length, password complexity and password topologies should properly be included in the Password Policy. A "strong" password policy makes it difficult or even infeasible for one to guess the password through either manual or automated means.


#### Static Analysis

Regular Expressions are often used to validate passwords. The password verification check against a defined password policy need to be reviewed if it rejects passwords that violate the password policy.

Passwords can be set when registering accounts, changing the password or when resetting the password in a forgot password process. All of the available mechanisms in the application need to use the same password verification check that is aligned with the password policy.

If a frameworks is used that offers the possibility to create and enforce a password policy for all users of the application, the configuration should be checked.


#### Dynamic Analysis

All available functions that allow a user to set a password need to verified if passwords can be used that violate the password policy specifications. This can be:

- Self-registration function for new users that allows to specify a password
- Forgot Password function that allows a user to set a new password
- Change Password function that allows a logged in user to set a new password

An interception proxy should be used, to bypass local passwords checks within the app and to be able verify the password policy implemented on server side.


#### Remediation

A good password policy should define the following requirements in order to avoid password guessing attacks or even brute-forcing.

#####  Password Length
* Minimum length of the passwords should be enforced, at least 10 characters.
* Maximum password length should not be set too low, as it will prevent users from creating passphrases. Typical maximum length is 128 characters.

##### Password Complexity
* Password must meet at least 3 out of the following 4 complexity rules
1. at least 1 uppercase character (A-Z)
2. at least 1 lowercase character (a-z)
3. at least 1 digit (0-9)
4. at least 1 special character (punctuation)

For further details check the OWASP Authentication Cheat Sheet<sup>[1]</sup>.

#### References

##### OWASP Mobile Top 10 2016
* M4 - Insecure Authentication - https://www.owasp.org/index.php/Mobile_Top_10_2016-M4-Insecure_Authentication

##### OWASP MASVS
* 4.4: "A password policy exists and is enforced at the remote endpoint."

##### CWE
- CWE-521 - Weak Password Requirements

##### Info
* [1] OWASP Authentication Cheat Sheet - https://www.owasp.org/index.php/Authentication_Cheat_Sheet#Implement_Proper_Password_Strength_Controls
* [2] OWASP Testing Guide (OTG-AUTHN-007) - https://www.owasp.org/index.php/Testing_for_Weak_password_policy_(OTG-AUTHN-007)


### Testing Excessive Login Attempts

#### Overview

We all have heard about brute force attacks. This is one of the simplest attack types, as already many tools are available that work out of the box. It also doesn’t require a deep technical understanding of the target, as only a list of username and password combinations is sufficient to execute the attack. Once a valid combination of credentials is identified access to the application is possible and the account can be compromised.
 
To be protected against these kind of attacks, applications need to implement a control to block the access after a defined number of incorrect login attempts.
 
Depending on the application that you want to protect, the number of incorrect attempts allowed may vary. For example, in a banking application it should be around three to five attempts, but, in a public forum, it could be a higher number. Once this threshold is reached it also needs to be decided if the account gets locked permanently or temporarily. Locking the account temporarily is also called login throttling.
 
The test consists by entering the password incorrectly for the defined number of attempts to trigger the account lockout. At that point, the anti-brute force control should be activated and your logon should be rejected when the correct credentials are entered.

#### Static Analysis

It need to be checked that a validation method exists during logon that checks if the number of attempts for a username equals to the maximum number of attempts set. In that case, no logon should be granted once this threshold is meet. After a correct attempt, there should also be a mechanism in place to set the error counter to zero.


#### Dynamic Analysis

For a dynamic analysis of the application an interception proxy should be used. The following steps can be applied to check if the lockout mechanism is implemented properly.  
1.  Log in incorrectly for a number of times to trigger the lockout control (generally 3 to 15 incorrect attempts)
2.  Once you have locked out the account, enter the correct logon details to verify if login is not possible anymore.
If this is correctly implemented logon should be denied when the right password is entered, as the credential has already been blocked.

#### Remediation

Lockout controls have to be implemented on server side to prevent brute force attacks. Further mitigation techniques are described by OWASP in Blocking Brute Force Attacks<sup>[3]</sup>.
It is interesting to clarify that incorrect login attempts should be cumulative and not linked to a session. If you implement a control to block the credential in your 3rd attempt in the same session, it can be easily bypassed by entering the details wrong two times and get a new session. This will then give another two free attempts.

Alternatives to locking accounts are enforcing 2-Factor-Authentication (2FA) for all accounts or the usage of CAPTCHAS. See also Credential Cracking OAT-007 in the OWASP Automated Thread Handbook<sup>[4]</sup>.

#### References

##### OWASP Mobile Top 10 2016
* M4 - Insecure Authentication - https://www.owasp.org/index.php/Mobile_Top_10_2016-M4-Insecure_Authentication

##### OWASP MASVS
* 4.5: "The remote endpoint implements an exponential back-off, or temporarily locks the user account, when incorrect authentication credentials are submitted an excessive number of times ."

##### CWE

-- TODO [Add relevant CWE for "Testing Excessive Login Attempts"] --
- CWE-312 - Cleartext Storage of Sensitive Information

##### Info
* [1] OTG-AUTHN-003 - https://www.owasp.org/index.php/Testing_for_Weak_lock_out_mechanism
* [2] Brute Force Attacks - https://www.owasp.org/index.php/Brute_force_attack
* [3] Blocking Brute Force Attacks - https://www.owasp.org/index.php/Blocking_Brute_Force_Attacks
* [4] OWASP Automated Threats to Web Applications - https://www.owasp.org/index.php/OWASP_Automated_Threats_to_Web_Applications

##### Tools
* Burp Suite Professional - https://portswigger.net/burp/
* OWASP ZAP - https://www.owasp.org/index.php/OWASP_Zed_Attack_Proxy_Project



### Testing the Session Timeout

#### Overview

Compared to web applications most mobile applications don’t have a session timeout mechanism that terminates the session after some period of inactivity and force the user to login again. For most mobile applications users need to enter the credentials once. After authenticating on server side an access token is stored on the device which is used to authenticate. If the token is about to expire the token will be renewed transparently without entering the credentials again (e.g. OAuth2 or JWT). Applications that handle sensitive data like patient data or critical functions like financial transactions should implement a session timeout as a security-in-depth measure that forces users to re-login after a defined period.
 
We will explain here how to check that this control is implemented correctly, both in the client and server side.

#### Static Analysis

If server side code is available, it should be reviewed that the session timeout functionality is correctly configured and a timeout is triggered after a defined period of time.  
The check needed here will be different depending on the technology used. Here are different examples on how a session timeout can be configured:
- Spring (Java) - http://docs.spring.io/spring-session/docs/current/reference/html5/
- Ruby on Rails -  https://github.com/rails/rails/blob/318a20c140de57a7d5f820753c82258a3696c465/railties/lib/rails/application/configuration.rb#L130
- PHP - http://php.net/manual/en/session.configuration.php#ini.session.gc-maxlifetime
- ASP.Net - https://msdn.microsoft.com/en-GB/library/system.web.sessionstate.httpsessionstate.timeout(v=vs.110).aspx
- Amazon AWS - http://docs.aws.amazon.com/ElasticLoadBalancing/latest/DeveloperGuide/config-idle-timeout.html
 

#### Dynamic Analysis

Dynamic analysis is an efficient option, as it is easy to validate if the session timeout is working or not at runtime using an interception proxy. This is similar to test case "Testing the Logout Functionality", but we need to leave the application in idle for the period of time required to trigger the timeout function. Once this condition has been launched, we need to validate that the session is effectively terminated on client and server side.

The following steps can be applied to check if the session timeout is implemented properly.  
1. Log into the application.
2. Do a couple of operations that require authentication inside the application.
3. Leave the application in idle until the session expires (for testing purposes, a reasonable timeout can be configured, and amended later in the final version)
 
Resend one of the operations executed in step 2 using an interception proxy, for example with Burp Repeater. The purpose of this is to send to the server a request with the session ID that has been invalidated when the session has expired.
If session timeout has been correctly configured on the server side, either an error message or redirect to the login page will be sent back to the client. On the other hand, if you have the same response you had in step 2, then, this session is still valid, which means that the session timeout is not configured correctly.
More information can also be found in the OWASP Web Testing Guide (OTG-SESS-007)<sup>[1]</sup>.

#### Remediation

Most of the frameworks have a parameter to configure the session timeout. This parameter should be set accordingly to the best practices specified of the documentation of the framework. The best practice timeout setting may vary between 10 minutes to two hours, depending on the sensitivity of your application and the use case of it.

#### References

##### OWASP Mobile Top 10 2016
* M4 - Insecure Authentication - https://www.owasp.org/index.php/Mobile_Top_10_2016-M4-Insecure_Authentication

##### OWASP MASVS
* 4.7: "Sessions are terminated at the remote endpoint after a predefined period of inactivity."

##### CWE
- CWE-613 - Insufficient Session Expiration

##### Info
* [1] OWASP Web Application Test Guide (OTG-SESS-007) -  https://www.owasp.org/index.php/Test_Session_Timeout_(OTG-SESS-007)
* [2] OWASP Session management cheatsheet https://www.owasp.org/index.php/Session_Management_Cheat_Sheet


### Testing 2-Factor Authentication

#### Overview

https://authy.com/blog/security-of-sms-for-2fa-what-are-your-options/
-- TODO [Provide a general description of the issue "Testing 2-Factor Authentication".] --

#### Static Analysis

-- TODO [Describe how to assess this given either the source code or installer package (APK/IPA/etc.), but without running the app. Tailor this to the general situation (e.g., in some situations, having the decompiled classes is just as good as having the original source, in others it might make a bigger difference). If required, include a subsection about how to test with or without the original sources.] --

-- TODO [Confirm remark on "Use the &lt;sup&gt; tag to reference external sources, e.g. Meyer's recipe for tomato soup<sup>[1]</sup>."] --

-- TODO [Develop content on Testing 2-Factor Authentication with source code] --


#### Dynamic Analysis

-- TODO [Describe how to test for this issue "Testing 2-Factor Authentication" by running and interacting with the app. This can include everything from simply monitoring network traffic or aspects of the app’s behavior to code injection, debugging, instrumentation, etc.] --

#### Remediation

-- TODO [Describe the best practices that developers should follow to prevent this issue "Testing 2-Factor Authentication".] --

#### References

##### OWASP Mobile Top 10 2016
* M4 - Insecure Authentication - https://www.owasp.org/index.php/Mobile_Top_10_2016-M4-Insecure_Authentication

##### OWASP MASVS
* 4.8: "A second factor of authentication exists at the remote endpoint and the 2FA requirement is consistently enforced."

##### CWE

-- TODO [Add relevant CWE for "Testing 2-Factor Authentication"] --
- CWE-312 - Cleartext Storage of Sensitive Information

##### Info

- [1] Meyer's Recipe for Tomato Soup - http://www.finecooking.com/recipes/meyers-classic-tomato-soup.aspx
- [2] Another Informational Article - http://www.securityfans.com/informational_article.html

##### Tools

-- TODO [Add relevant tools for "Testing 2-Factor Authentication"] --
* Enjarify - https://github.com/google/enjarify




### Testing Step-up Authentication

#### Overview

-- TODO [Provide a general description of the issue "Testing Step-up Authentication".] --

#### Static Analysis

-- TODO [Describe how to assess this given either the source code or installer package (APK/IPA/etc.), but without running the app. Tailor this to the general situation (e.g., in some situations, having the decompiled classes is just as good as having the original source, in others it might make a bigger difference). If required, include a subsection about how to test with or without the original sources.] --

-- TODO [Confirm remark on "Use the &lt;sup&gt; tag to reference external sources, e.g. Meyer's recipe for tomato soup<sup>[1]</sup>." ] --

-- TODO [Develop content on Testing Step-up Authentication with source code] --

#### Dynamic Analysis

-- TODO [Describe how to test for this issue "Testing Step-up Authentication" by running and interacting with the app. This can include everything from simply monitoring network traffic or aspects of the app’s behavior to code injection, debugging, instrumentation, etc.] --

#### Remediation

-- TODO [Describe the best practices that developers should follow to prevent this issue "Testing Step-up Authentication".] --

#### References

##### OWASP Mobile Top 10 2016
* M4 - Insecure Authentication - https://www.owasp.org/index.php/Mobile_Top_10_2016-M4-Insecure_Authentication

##### OWASP MASVS
* 4.9: "Step-up authentication is required to enable actions that deal with sensitive data or transactions."

##### CWE

-- TODO [Add relevant CWE for "Testing Step-up Authentication"] --
- CWE-312 - Cleartext Storage of Sensitive Information

##### Info

- [1] Meyer's Recipe for Tomato Soup - http://www.finecooking.com/recipes/meyers-classic-tomato-soup.aspx
- [2] Another Informational Article - http://www.securityfans.com/informational_article.html

##### Tools

-- TODO [Add relevant tools for "Testing Step-up Authentication"] --
* Enjarify - https://github.com/google/enjarify


### Testing User Device Management

#### Overview

-- TODO [Provide a general description of the issue "Testing User Device Management".] --

#### Static Analysis

-- TODO [Describe how to assess this given either the source code or installer package (APK/IPA/etc.), but without running the app. Tailor this to the general situation (e.g., in some situations, having the decompiled classes is just as good as having the original source, in others it might make a bigger difference). If required, include a subsection about how to test with or without the original sources.] --

-- TODO [Confirm remark on "Use the &lt;sup&gt; tag to reference external sources, e.g. Meyer's recipe for tomato soup<sup>[1]</sup>."] --

--TODO [Develop content on Testing User Device Management with source code] --


#### Dynamic Analysis

-- TODO [Describe how to test for this issue "Testing User Device Management" by running and interacting with the app. This can include everything from simply monitoring network traffic or aspects of the app’s behavior to code injection, debugging, instrumentation, etc.] --

#### Remediation

-- TODO [Describe the best practices that developers should follow to prevent this issue "Testing User Device Management".] --

#### References

##### OWASP Mobile Top 10 2016
* M4 - Insecure Authentication - https://www.owasp.org/index.php/Mobile_Top_10_2016-M4-Insecure_Authentication

##### OWASP MASVS
* 4.10: "The app informs the user of all login activities with his or her account. Users are able view a list of devices used to access the account, and to block specific devices."

##### CWE

-- TODO [Add relevant CWE for "Testing User Device Management"] --
- CWE-312 - Cleartext Storage of Sensitive Information

##### Info

- [1] Meyer's Recipe for Tomato Soup - http://www.finecooking.com/recipes/meyers-classic-tomato-soup.aspx
- [2] Another Informational Article - http://www.securityfans.com/informational_article.html

##### Tools

-- TODO [Add relevant tools for "Testing User Device Management"] --
* Enjarify - https://github.com/google/enjarify
