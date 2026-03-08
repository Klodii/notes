# Introduction to Web Applications

Web applications are interactive applications that run on web browsers. Web
applications usually adopt a client-server architecture to run and handle
interactions. They typically have front end components (i.e., the website
interface, or "what the user sees") that run on the client-side (browser) and
other back end components (web application source code) that run on the
server-side (back end server/databases).

## Websites vs. Web Applications

### Websites:
- static pages
- same for everyone
- Web 1.0

### Web Applications
- dynamic pages
- interactive
- different view for each user
- functional
- Web 2.0

## Security Risks of Web Applications

To properly pentest web applications, we need to understand how they work, how
they are developed, and what kind of risk lies at each layer and component of
the application depending on the technologies in use.


We will always come across various web applications that are designed and
configured differently. One of the most current and widely used methods for
testing web applications is the OWASP Web Security Testing Guide.


One of the most common procedures is to start by reviewing a web application's
front end components, such as HTML, CSS and JavaScript (also known as the front
end trinity), and attempt to find vulnerabilities such as Sensitive Data
Exposure and Cross-Site Scripting (XSS). Once all front end components are
thoroughly tested, we would typically review the web application's core
functionality and the interaction between the browser and the webserver to
enumerate the technologies the webserver uses and look for exploitable flaws. We
typically assess web applications from both an unauthenticated and authenticated
perspective (if the application has login functionality) to maximize coverage
and review every possible attack scenario.


Web applications provide a vast attack surface, and their dynamic nature means
that they are constantly changing (and overlooked!). A relatively simple code
change may introduce a catastrophic vulnerability or a series of vulnerabilities
that can be chained together to gain access to sensitive data or remote code
execution on the webserver or other hosts in the environment, such as database
servers.

A well-rounded infosec professional should have a deep understanding of web
applications and be as comfortable attacking web applications as performing
network penetration testing and Active Directory attacks. A penetration tester
with a strong foundation in web applications can often set themselves apart from
their peers and find flaws that others may overlook


| Flaw                                       | Real-world Scenario |
|--------------------------------------------|---------------------|
| SQL injection                              | Obtaining Active Directory usernames and performing a password spraying attack against a VPN or email portal. |
| File Inclusion                             | Reading source code to find a hidden page or directory which exposes additional functionality that can be used to gain remote code execution. |
| Unrestricted File Upload  | A web application that allows a user to upload a profile picture that allows any file type to be uploaded (not just images). This can be leveraged to gain full control of the web application server by uploading malicious code. |
| Insecure Direct Object Referencing (IDOR)  | When combined with a flaw such as broken access control, this can often be used to access another user's files or functionality. An example would be editing your user profile browsing to a page such as /user/701/edit-profile. If we can change the 701 to 702, we may edit another user's profile! |
| Broken Access Control                      | Another example is an application that allows a user to register a new account. If the account registration functionality is designed poorly, a user may perform privilege escalation when registering. Consider the POST request when registering a new user, which submits the data username=bjones&password=Welcome1&email=bjones@inlanefreight.local&roleid=3. What if we can manipulate the roleid parameter and change it to 0 or 1. We have seen real-world applications where this was the case, and it was possible to quickly register an admin user and access many unintended features of the web application. |

## Web Application Components


1. Client
2. Server
   - Webserver
   - Web Application Logic
   - Database
3. Services (Microservices)
    - 3rd Party Integrations
    - Web Application Integrations
4. Functions (Serverless)

## Top 20 most common mistakes web developers make

1. Permitting Invalid Data to Enter the Database
2. Focusing on the System as a Whole
3. Establishing Personally Developed Security Methods
4. Treating Security to be Your Last Step
5. Developing Plain Text Password Storage
6. Creating Weak Passwords
7. Storing Unencrypted Data in the Database
8. Depending Excessively on the Client Side
9. Being Too Optimistic
10. Permitting Variables via the URL Path Name
11. Trusting third-party code
12. Hard-coding backdoor accounts
13. Unverified SQL injections
14. Remote file inclusions
15. Insecure data handling
16. Failing to encrypt data properly
17. Not using a secure cryptographic system
18. Ignoring layer 8
19. Review user actions
20. Web Application Firewall misconfigurations

These mistakes lead to the OWASP Top 10 vulnerabilities for web applications

1. Broken Access Control
2. Cryptographic Failures
3. Injection
4. Insecure Design
5. Security Misconfiguration
6. Vulnerable and Outdated Components
7. Identification and Authentication Failures
8. Software and Data Integrity Failures
9. Security Logging and Monitoring Failures
10. Server-Side Request Forgery (SSRF)


## Front End Vulnerabilities

### Sensitive Data Exposure

It is always important to review the code that will be visible to end-users
through the page source (Ctrl-U) or run it through tools to check for exposed
information.

### HTMP Injection
Caused by Unfiletered user input (no validation on input)

HTML injection occurs when unfiltered user input is displayed on the page. This
can either be through retrieving previously submitted code, like retrieving a
user comment from the back end database, or by directly displaying unfiltered
user input through JavaScript on the front end.

When a user has complete control of how their input will be displayed, they can
submit HTML code, and the browser may display it as part of the page. This may
include a malicious HTML code, like an external login form, which can be used to
trick users into logging in while actually sending their login credentials to a
malicious server to be collected for other attacks.

Another example of HTML Injection is web page defacing. This consists of
injecting new HTML code to change the web page's appearance, inserting malicious
ads, or even completely changing the page. This type of attack can result in
severe reputational damage to the company hosting the web application.

### Cross-Site Scripting (XSS)
Caused by Unfiletered user input (no validation on input)

HTML Injection vulnerabilities can often be utilized to also perform Cross-Site
Scripting (XSS) attacks by injecting JavaScript code to be executed on the
client-side. Once we can execute code on the victim's machine, we can
potentially gain access to the victim's account or even their machine. XSS is
very similar to HTML Injection in practice. However, XSS involves the injection
of JavaScript code to perform more advanced attacks on the client-side, instead
of merely injecting HTML code.


There are three main types of XSS:

- **Reflected XSS**: Occurs when user input is displayed on the page after processing (e.g., search result or error message).
- **Stored XSS**: Occurs when user input is stored in the back end database and then displayed upon retrieval (e.g., posts or comments).
- **DOM XSS**: Occurs when user input is directly shown in the browser and is written to an HTML DOM object (e.g., vulnerable username or page title).

### Cross-Site Request Forgery (CSRF)
Caused by Unfiletered user input (no validation on input)

The third type of front end vulnerability that is caused by unfiltered user
input is Cross-Site Request Forgery (CSRF). CSRF attacks may utilize XSS
vulnerabilities to perform certain queries, and API calls on a web application
that the victim is currently authenticated to. This would allow the attacker to
perform actions as the authenticated user. It may also utilize other
vulnerabilities to perform the same functions, like utilizing HTTP parameters
for attacks.

A common CSRF attack to gain higher privileged access to a web application is to
craft a JavaScript payload that automatically changes the victim's password to
the value set by the attacker. Once the victim views the payload on the
vulnerable page (e.g., a malicious comment containing the JavaScript CSRF
payload), the JavaScript code would execute automatically. It would use the
victim's logged-in session to change their password. Once that is done, the
attacker can log in to the victim's account and control it.


#### Prevention

- **Sanitization**: Removing special characters and non-standard characters from user input before displaying it or storing it.
- **Validation**: Ensuring that submitted user input matches the expected format (i.e., submitted email matched email format)
- HTTP-level defenses like the SameSite cookie attribute (SameSite=Strict or Lax)
can restrict browsers from including authentication cookies in cross-origin
requests

## Back End Vulnerabilities

### Broken Authentication/Access Control

Broken authentication and Broken Access Control are among the most common and
most dangerous vulnerabilities for web applications.

Broken Authentication refers to vulnerabilities that allow attackers to bypass
authentication functions. For example, this may allow an attacker to login
without having a valid set of credentials or allow a normal user to become an
administrator without having the privileges to do so.

Broken Access Control refers to vulnerabilities that allow attackers to access
pages and features they should not have access to. For example, a normal user
gaining access to the admin panel.

### Malicious File Upload

Another common way to gain control over web applications is through uploading
malicious scripts. If the web application has a file upload feature and does not
properly validate the uploaded files, we may upload a malicious script (i.e., a
PHP script), which will allow us to execute commands on the remote server.

For example, the WordPress Plugin Responsive Thumbnail Slider 1.0 can be
exploited to upload any arbitrary file, including malicious scripts, by
uploading a file with a double extension (i.e. shell.php.jpg). There's even a
Metasploit Module that allows us to exploit this vulnerability easily.

### Command Injection

Many web applications execute local Operating System commands to perform certain
processes. For example, a web application may install a plugin of our choosing
by executing an OS command that downloads that plugin, using the plugin name
provided. If not properly filtered and sanitized, attackers may be able to
inject another command to be executed alongside the originally intended command
(i.e., as the plugin name), which allows them to directly execute commands on
the back end server and gain control over it. This type of vulnerability is
called command injection.

This vulnerability is widespread, as developers may not properly sanitize user
input or use weak tests to do so, allowing attackers to bypass any checks or
filtering put in place and execute their commands.

For example, the WordPress Plugin Plainview Activity Monitor 20161228 has a
vulnerability that allows attackers to inject their command in the ip value, by
simply adding | COMMAND... after the ip value.

### SQL Injection (SQLi)

Another very common vulnerability in web applications is a SQL Injection
vulnerability. Similarly to a Command Injection vulnerability, this
vulnerability may occur when the web application executes a SQL query, including
a value taken from user-supplied input.
