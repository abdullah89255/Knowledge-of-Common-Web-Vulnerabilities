# Knowledge-of-Common-Web-Vulnerabilities

**web security / bug bounty**, this means being able to recognize common vulnerability classes, understand **why they happen**, understand their impact, and know how to validate them safely in an authorized environment.

A useful roadmap is:

```text
Web Vulnerabilities
│
├── Injection
│   ├── SQL Injection
│   ├── Command Injection
│   ├── SSTI
│   └── NoSQL Injection
│
├── Client-Side
│   ├── XSS
│   ├── DOM XSS
│   ├── CSRF
│   └── Clickjacking
│
├── Access Control
│   ├── IDOR / BOLA
│   ├── Privilege Escalation
│   └── Authentication flaws
│
├── Server-Side
│   ├── SSRF
│   ├── XXE
│   ├── Path Traversal
│   ├── File Inclusion
│   └── Deserialization
│
├── API Security
│   ├── Broken Authorization
│   ├── Mass Assignment
│   ├── Excessive Data Exposure
│   └── Rate Limiting
│
└── Configuration / Logic
    ├── CORS Misconfiguration
    ├── Security Misconfiguration
    ├── Information Disclosure
    ├── Open Redirect
    └── Business Logic Bugs
```

---

# 1. XSS — Cross-Site Scripting

XSS happens when an application handles untrusted input in a way that causes the browser to interpret it as executable content.

### Example

Suppose a website displays:

```text
Welcome, Mamun
```

The application receives the username from a user-controlled source.

Conceptually:

```text
User input
    ↓
Application
    ↓
HTML
    ↓
Browser
```

If the application doesn't properly encode the output, attacker-controlled markup/script can potentially execute in another user's browser.

### Types

**Reflected XSS**

```text
Request
  ↓
Server
  ↓
Response
  ↓
Browser
```

**Stored XSS**

```text
Input
 ↓
Database
 ↓
Other users
 ↓
Browser
```

**DOM XSS**

The vulnerability occurs primarily in client-side JavaScript/DOM manipulation.

### Learn

* HTML encoding
* JavaScript contexts
* DOM sources
* DOM sinks
* `innerHTML`
* `textContent`
* Content Security Policy

---

# 2. SQL Injection

SQL injection occurs when untrusted input changes the intended structure or meaning of a database query.

Conceptually:

```text
User input
    ↓
Application
    ↓
SQL query
    ↓
Database
```

Unsafe application design can allow user-controlled input to influence SQL syntax.

### Example concept

Imagine an application internally constructs:

```sql
SELECT * FROM users WHERE username = '<input>';
```

The secure solution is generally to use **parameterized queries/prepared statements**, rather than concatenating untrusted input into SQL.

### Learn

* SQL fundamentals
* SELECT
* INSERT
* UPDATE
* DELETE
* WHERE
* JOIN
* Prepared statements
* Error handling
* Blind SQL injection concepts

Practice SQL injection in deliberately vulnerable labs rather than against systems without permission.

---

# 3. IDOR / BOLA

This is one of the **most important vulnerabilities for bug bounty**.

**IDOR = Insecure Direct Object Reference**

**BOLA = Broken Object Level Authorization**

Suppose an API has:

```http
GET /api/orders/1001
```

A user is authorized to access order `1001`.

The important security question is:

> Does the server verify that this particular user is authorized to access the requested object?

Changing an identifier alone isn't enough to establish a vulnerability.

The secure architecture is:

```text
Request
 ↓
Authentication
 ↓
Identify user
 ↓
Check authorization for object
 ↓
Return object
```

### Learn to distinguish

```text
Authentication
= Who are you?

Authorization
= What are you allowed to access?
```

---

# 4. CSRF

**Cross-Site Request Forgery** involves tricking a user's browser into making an unwanted authenticated request when the application lacks adequate protections.

Example scenario:

```text
Victim logged into website
          ↓
Visits another website
          ↓
Browser makes request to target
          ↓
Target incorrectly accepts request
```

Important defenses include:

* CSRF tokens
* SameSite cookies
* Origin validation
* Appropriate authentication design

### Important

CSRF generally concerns **actions performed using the victim's authenticated context**. Modern browser cookie policies can reduce some CSRF scenarios, but developers still need proper defenses.

---

# 5. Authentication Vulnerabilities

Authentication answers:

> Who is this user?

Common problems include:

* Weak password policies
* Account enumeration
* Poor session management
* Broken password-reset flows
* Missing MFA protections
* Session fixation
* Session token weaknesses
* Improper logout/invalidation
* Authentication bypass

### Example

A password-reset system might have:

```text
Forgot password
       ↓
Email
       ↓
Reset token
       ↓
New password
```

Security researchers examine whether the complete flow properly verifies identity and protects the reset token.

---

# 6. Authorization Vulnerabilities

Authorization problems occur when a user can perform an action they shouldn't be allowed to perform.

Example:

```text
Normal user
    ↓
/admin/settings
    ↓
Application
    ↓
Should reject
```

But if the application incorrectly permits the operation, that's a security issue.

### Types

* Horizontal privilege escalation

```text
User A → User B's resources
```

* Vertical privilege escalation

```text
Normal User → Admin functionality
```

Authorization is often more subtle than authentication.

---

# 7. SSRF — Server-Side Request Forgery

SSRF occurs when an application makes server-side network requests using attacker-controlled input.

Conceptually:

```text
Attacker
   ↓
Web Application
   ↓
Server makes request
   ↓
Internal/external resource
```

Potentially affected applications include those with features such as:

```text
URL preview
Webhook configuration
Image import
Remote document fetching
URL validation
```

### Security concepts

Learn:

* Internal vs external networks
* URL parsing
* Network boundaries
* DNS
* HTTP clients
* Cloud metadata concepts
* SSRF defenses

SSRF can be particularly serious because the vulnerable server may have network access that the external user doesn't.

---

# 8. Path Traversal

Path traversal occurs when user-controlled input allows access outside the intended filesystem directory.

Conceptually:

```text
Application
    ↓
Requested file
    ↓
Filesystem
```

The application should ensure that a user cannot escape the intended directory.

For example, a file-download feature might conceptually accept:

```text
/download?file=report.pdf
```

The security question is whether the application properly constrains what filesystem resources can be requested.

### Learn

* Filesystem paths
* Absolute vs relative paths
* Canonicalization
* File permissions
* Secure file handling

---

# 9. File Upload Vulnerabilities

Consider:

```text
User
 ↓
Upload file
 ↓
Web server
 ↓
Storage
```

Security problems can occur when an application doesn't properly validate uploaded files.

Important controls include:

* File type validation
* Content validation
* Size limits
* Safe filenames
* Storage outside executable web directories
* Access controls
* Malware scanning where appropriate

Don't rely only on the filename extension or client-provided MIME type.

---

# 10. Command Injection

Command injection can occur when an application passes untrusted input into an operating-system command.

Conceptually:

```text
User input
    ↓
Application
    ↓
OS command
    ↓
Operating system
```

For example, an application might have a feature that invokes a system utility.

The secure approach is to avoid shell execution where possible and use safe APIs with strict input validation.

### Learn

* Processes
* Shells
* Argument parsing
* OS permissions
* Safe process execution

This is especially important for applications written in:

```text
PHP
Python
Node.js
Java
.NET
```

---

# 11. SSTI — Server-Side Template Injection

SSTI occurs when user-controlled input is interpreted as a server-side template rather than ordinary data.

Architecture:

```text
User input
    ↓
Template engine
    ↓
Server
    ↓
Generated response
```

Common template technologies include:

```text
Jinja2       → Python
Twig         → PHP
Freemarker   → Java
Thymeleaf    → Java
Razor        → .NET
```

The key distinction is:

```text
Data
vs.
Template/code
```

---

# 12. XXE — XML External Entity

XXE involves insecure XML parsing.

Conceptually:

```text
XML input
   ↓
XML parser
   ↓
Application
```

If dangerous XML features are enabled unnecessarily, attackers may potentially cause the server to access unintended resources.

Learn:

* XML
* DTD
* External entities
* XML parsers
* Secure parser configuration

---

# 13. CORS Misconfiguration

CORS controls which origins browser JavaScript can interact with for cross-origin requests.

Example:

```http
Access-Control-Allow-Origin: https://example.com
```

Potential security problems occur when sensitive resources are exposed to inappropriate origins, especially when credentials are involved.

Understand:

```text
Same-Origin Policy
        ↓
CORS
        ↓
Browser decides whether JS can read response
```

Remember:

**CORS is not authentication.**

A server must still enforce authentication and authorization independently.

---

# 14. Open Redirect

An open redirect occurs when a website allows an attacker-controlled destination to be used for redirection without adequate validation.

Conceptually:

```text
example.com/redirect?url=...
        ↓
Application
        ↓
External destination
```

Open redirects can be abused in phishing and authentication flows and can sometimes become more significant when combined with other vulnerabilities.

---

# 15. Security Misconfiguration

This is a broad category.

Examples include:

```text
Debug mode enabled
Unnecessary services
Default credentials
Excessive permissions
Verbose errors
Incorrect CORS
Missing security headers
Exposed administration interfaces
Improper cloud configuration
```

For example, an application might accidentally expose detailed internal error information:

```text
Database connection failed
/path/to/application/config.py
```

That can become an **information disclosure** issue.

---

# 16. Information Disclosure

The application accidentally reveals information that shouldn't be public.

Examples:

```text
Stack traces
Internal IP addresses
API keys
Debug information
Source-code fragments
User information
Internal filenames
Database errors
Technology versions
```

Not every piece of disclosed information is a vulnerability.

The important questions are:

```text
What information?
Who can access it?
Was it intended to be public?
What security impact does it create?
```

---

# 17. API Security Vulnerabilities

Modern applications heavily depend on APIs.

Typical architecture:

```text
Browser
   ↓
JavaScript
   ↓
API
   ↓
Backend
   ↓
Database
```

Important API vulnerabilities include:

### BOLA

```text
/api/users/123
```

Improper object authorization.

### Broken Function-Level Authorization

A normal user accesses functionality intended only for privileged users.

### Mass Assignment

The application accepts fields that users shouldn't be able to modify.

For example, an API may legitimately allow:

```json
{
    "name": "Mamun"
}
```

but accidentally process security-sensitive fields such as:

```json
{
    "name": "Mamun",
    "role": "admin"
}
```

The vulnerability depends on whether the server improperly permits unauthorized modification.

### Excessive Data Exposure

API returns more information than the client actually needs.

---

# 18. Race Conditions

A race condition occurs when security depends on timing or ordering between concurrent operations.

Example concept:

```text
Request A ───────┐
                 ├── Server
Request B ───────┘
```

If the application checks and updates state incorrectly, simultaneous requests may produce an unintended result.

Potential areas include:

* Coupon redemption
* Money transfers
* Inventory
* Account changes
* One-time actions
* Password/reset operations

These are often more advanced bugs.

---

# 19. Business Logic Vulnerabilities

This is one of the most valuable areas for experienced bug bounty researchers.

A business-logic vulnerability occurs when the application technically works as programmed but the **business rules can be violated**.

Example:

```text
Product price = $100
Quantity = 1
```

The intended process might be:

```text
Add product
 ↓
Calculate price
 ↓
Apply valid discount
 ↓
Pay
```

A logic flaw might allow an unintended sequence of operations.

The key question isn't:

> "Can I inject something?"

It's:

> **"Can I make the application do something the business rules didn't intend?"**

---

# 20. Prototype Pollution

Since you're interested in JavaScript security, this deserves special attention.

JavaScript objects inherit properties through prototypes.

Prototype pollution can occur when an application unsafely merges or modifies attacker-influenced object properties.

Conceptually:

```text
User-controlled object
        ↓
Unsafe merge/update
        ↓
Object prototype
        ↓
Unexpected properties
        ↓
Application behavior changes
```

It can become especially interesting in Node.js applications.

Study:

```text
Objects
Prototypes
Prototype chain
Object inheritance
Property lookup
Object.assign()
Deep merge functions
JSON parsing
Node.js behavior
```

---

# Vulnerability Severity

Don't judge a vulnerability only by its name.

For example:

```text
XSS
```

could be:

```text
Low impact
```

in one context and much more serious in another.

Similarly:

```text
Information disclosure
```

could be insignificant or highly sensitive.

Think:

```text
Vulnerability
      ↓
Exploitability
      ↓
Affected users
      ↓
Data/functionality exposed
      ↓
Security impact
```

---

# How to Analyze a Web Application

When you encounter an authorized target, build a mental model first.

### Step 1 — Identify functionality

```text
Login
Register
Profile
Search
Upload
Payment
Settings
Admin
API
```

### Step 2 — Identify requests

Use browser DevTools or Burp Suite:

```text
GET
POST
PUT
PATCH
DELETE
```

### Step 3 — Identify inputs

```text
Query parameters
Form fields
JSON
Headers
Cookies
Path parameters
File uploads
```

### Step 4 — Identify trust boundaries

Ask:

```text
What does the browser control?
What does the server control?
What does the API trust?
What does the database trust?
```

### Step 5 — Check security decisions

```text
Authentication
       ↓
Authorization
       ↓
Input validation
       ↓
Business logic
       ↓
Data access
```

---

# The Most Important Vulnerabilities to Master First

If you're starting web security, I recommend this order:

```text
01. HTTP fundamentals
        ↓
02. XSS
        ↓
03. SQL Injection
        ↓
04. Authentication
        ↓
05. Authorization
        ↓
06. IDOR / BOLA
        ↓
07. CSRF
        ↓
08. File Upload
        ↓
09. Path Traversal
        ↓
10. SSRF
        ↓
11. CORS
        ↓
12. JWT/API security
        ↓
13. SSTI
        ↓
14. Prototype Pollution
        ↓
15. Race Conditions
        ↓
16. Business Logic
```

### Your ultimate goal

Don't become someone who simply knows:

```text
"Nmap command"
"Nuclei command"
"Burp button"
"XSS payload"
```

Aim to become someone who can look at:

```http
POST /api/account/update
Authorization: Bearer ...
Content-Type: application/json

{
    "name": "...",
    "email": "..."
}
```

and immediately start asking:

```text
Who is authenticated?
        ↓
What are they authorized to modify?
        ↓
Which fields are actually accepted?
        ↓
How is input validated?
        ↓
What happens server-side?
        ↓
What does the response reveal?
        ↓
Does the operation obey the application's business rules?
```

# These are some of the most important vulnerability classes for **web application security and bug bounty testing**. 

* What it is
* How it happens
* Simple example
* What to look for during testing
* How developers prevent it
* Bug-bounty impact

> **Important:** Only test systems you own or systems where you have explicit authorization. The examples below are designed for understanding and controlled labs.

---

# 1. Authentication Flaws

### What is authentication?

Authentication answers:

> **“Who are you?”**

Examples:

* Username + password
* OTP
* MFA
* Passkeys
* OAuth
* SSO
* API tokens

### Common authentication flaws

#### A. Weak password policy

For example:

```text
Username: admin
Password: admin123
```

If the application allows extremely weak passwords, accounts may be easier to compromise.

#### B. User enumeration

Suppose:

```http
POST /login
```

With an invalid username:

```json
{
  "error": "User does not exist"
}
```

But with an existing username:

```json
{
  "error": "Incorrect password"
}
```

An attacker can determine which usernames exist.

Better:

```json
{
  "error": "Invalid username or password"
}
```

#### C. Missing rate limiting

A login endpoint that accepts unlimited attempts can be vulnerable to password guessing.

Look at:

```text
POST /login
POST /api/auth/login
POST /forgot-password
POST /verify-otp
```

### Other authentication problems

* Password reset weaknesses
* OTP reuse
* OTP not expiring
* MFA bypass
* Session fixation
* Authentication state confusion
* Account recovery flaws
* OAuth implementation mistakes
* Remember-me token weaknesses

### Prevention

Use:

* Strong password hashing such as Argon2id/bcrypt
* Rate limiting
* MFA
* Secure password-reset tokens
* Short-lived OTPs
* Generic authentication errors
* Proper session invalidation

---

# 2. Authorization Flaws

Authentication:

> **Who are you?**

Authorization:

> **What are you allowed to do?**

This distinction is extremely important.

Imagine:

```text
User A → account ID 1001
User B → account ID 1002
```

User A sends:

```http
GET /api/account/1001
```

Normally:

```json
{
  "name": "User A",
  "email": "userA@example.com"
}
```

If User A changes:

```http
GET /api/account/1002
```

and receives User B's information, that's an authorization failure.

This is commonly called **BOLA/IDOR**.

### Types

#### Horizontal privilege escalation

User A accesses another normal user's data.

```text
User A → User B's data
```

#### Vertical privilege escalation

A normal user performs an administrator action.

```text
Normal User → Admin function
```

For example:

```http
POST /api/admin/delete-user
```

If the server only checks whether you're logged in but doesn't verify admin privileges, that's a serious flaw.

### Key testing question

For every sensitive request ask:

> **Does the server verify that this particular user is allowed to perform this particular action on this particular object?**

---

# 3. Session Vulnerabilities

A session allows a server to remember that you've authenticated.

Example:

```http
Set-Cookie: sessionid=abc123
```

Browser subsequently sends:

```http
Cookie: sessionid=abc123
```

### Common session vulnerabilities

#### Session fixation

An attacker somehow causes a victim to use a session identifier that was already known.

After successful authentication, the application should normally issue a **new session ID**.

#### Session not invalidated after logout

User:

```text
Login → session A
Logout
```

If session A still works, that's a problem.

#### Session remains valid after password change

Changing a password should generally invalidate old sessions, depending on the application's intended security model.

#### Weak session IDs

Bad:

```text
session=12345
session=12346
session=12347
```

Session identifiers should be unpredictable.

### Cookie security

Look for:

```http
Set-Cookie: session=...; Secure; HttpOnly; SameSite=Lax
```

Important flags:

| Flag     | Purpose                                 |
| -------- | --------------------------------------- |
| Secure   | Send cookie over HTTPS                  |
| HttpOnly | Prevent JavaScript from reading it      |
| SameSite | Helps control cross-site cookie sending |

---

# 4. Security Misconfiguration

This is extremely broad.

It occurs when a system is deployed with unsafe settings.

### Examples

#### Debug mode enabled

```text
DEBUG=True
```

An error might reveal:

```text
/database/config.py
/home/app/users.py
SECRET_KEY=...
```

#### Default credentials

For example:

```text
admin/admin
```

#### Directory listing

```text
/uploads/
    backup.zip
    old-config.json
    database.sql
```

#### Exposed administration panels

```text
/admin
/administrator
/management
```

### Other examples

* Unnecessary HTTP methods
* Verbose error messages
* Default application configurations
* Exposed `.git` repository
* Exposed backup files
* Missing security headers
* Incorrect CORS
* Public cloud storage
* Exposed monitoring interfaces

### Prevention

Use secure production configuration and regularly audit:

```text
Debug → OFF
Default credentials → changed
Directory listing → disabled
Sensitive files → protected
Admin interfaces → restricted
Error messages → sanitized
```

---

# 5. CORS Misconfiguration

CORS = **Cross-Origin Resource Sharing**.

Suppose:

```text
https://bank.example
```

has an API:

```text
https://api.bank.example
```

A browser normally restricts cross-origin JavaScript access.

The server can explicitly allow origins.

Example:

```http
Access-Control-Allow-Origin: https://trusted.example
```

### Dangerous configuration

```http
Access-Control-Allow-Origin: *
```

This is particularly important when sensitive data is exposed.

Another dangerous pattern is dynamically reflecting arbitrary origins while also allowing credentials.

Conceptually:

```http
Origin: https://attacker.example

Access-Control-Allow-Origin: https://attacker.example
Access-Control-Allow-Credentials: true
```

If the application also returns sensitive authenticated data, this can become serious.

### Important

CORS is **not authentication**.

This:

```http
Access-Control-Allow-Origin: *
```

doesn't mean:

> "Anyone can log into the application."

It means browsers may allow JavaScript from permitted origins to read responses.

---

# 6. Open Redirect

An application redirects users based on an attacker-controlled URL.

Example:

```text
https://example.com/redirect?url=https://google.com
```

If arbitrary destinations are accepted:

```text
https://example.com/redirect?url=https://attacker.example
```

the application redirects the victim.

### Why does it matter?

Open redirect can be useful in:

* Phishing
* OAuth attacks
* Trust abuse
* Authentication-flow attacks

### Better design

Allow only known destinations:

```text
/redirect?next=/dashboard
```

Instead of arbitrary external URLs.

---

# 7. File Upload Vulnerabilities

Applications frequently allow:

```text
Profile picture
Resume
Documents
Attachments
```

A dangerous upload implementation may trust the filename or MIME type.

Example:

```http
POST /upload
Content-Type: multipart/form-data
```

with:

```text
avatar.jpg
```

### Problems

* Executable files accepted
* Script files stored in executable directories
* MIME type trusted without validation
* Filename path manipulation
* Oversized files
* Malicious SVG/HTML
* Publicly accessible sensitive uploads
* Archive extraction vulnerabilities

### Secure architecture

Uploaded files should ideally:

1. Validate type
2. Validate size
3. Generate a random server-side filename
4. Store outside executable web directories
5. Restrict permissions
6. Scan where appropriate
7. Serve with safe content types
8. Prevent script execution

---

# 8. Path Traversal

Path traversal occurs when user input influences filesystem paths without proper validation.

Suppose:

```http
GET /download?file=report.pdf
```

The application internally does something like:

```text
/uploads/{file}
```

If the application doesn't safely constrain the path, traversal sequences can potentially escape the intended directory.

Conceptually:

```text
uploads/
    report.pdf
```

The attacker attempts to navigate:

```text
uploads/../something
```

### Potential impact

Depending on permissions and application behavior:

* Read sensitive files
* Access configuration
* Access source code
* Expose credentials

### Prevention

Use:

* Allowlists
* Canonicalization
* Safe filesystem APIs
* Random file identifiers
* Storage outside sensitive filesystem areas

Never assume:

```text
"../"
```

is the only representation that needs consideration.

---

# 9. Local File Inclusion — LFI

LFI is related to path traversal but is specifically about an application **including/processing a local file** based on attacker-controlled input.

Imagine:

```text
https://example.com/page?template=home
```

Server code conceptually does:

```text
include(template)
```

If arbitrary local files can be selected, an attacker may influence what the application loads.

### Difference

**Path traversal:**

> Accessing a file outside the intended directory.

**LFI:**

> Application includes/loads a local file as part of its processing.

LFI can sometimes become more severe if the application processes the included file as executable code, depending on the technology and configuration.

---

# 10. Server-Side Template Injection — SSTI

SSTI occurs when user-controlled input becomes part of a **template itself**, rather than simply being data rendered by a template.

Consider:

```text
Hello {{username}}
```

Safe design:

```text
username = user_input
```

The template engine treats the username as data.

Dangerous architecture:

```text
template = "Hello " + user_input
render(template)
```

Now template syntax supplied by the user may be interpreted by the server.

### Common template technologies

* Jinja2
* Twig
* Freemarker
* Thymeleaf
* Razor
* Handlebars

### Why SSTI can be serious

Depending on the template engine and configuration, impact may range from:

```text
Template evaluation
        ↓
Sensitive data access
        ↓
Application internals
        ↓
Potential code execution
```

### Prevention

Never concatenate untrusted input into templates.

Use:

```text
static template + user data
```

rather than:

```text
user input → template source
```

---

# 11. Command Injection

Command injection happens when an application passes attacker-controlled input into an operating-system command.

Conceptually:

```text
User input
     ↓
Application
     ↓
OS command
```

Example of unsafe architecture:

```python
os.system("some-command " + user_input)
```

The fundamental problem isn't the particular command—it is that untrusted input is being interpreted as part of a shell command.

### Better design

Use APIs that don't invoke a shell and pass arguments separately.

Conceptually:

```python
subprocess.run(
    ["some-command", user_value],
    shell=False
)
```

Even then, validate the input according to the application's requirements.

### Impact

Potentially:

* Read files
* Modify data
* Access internal resources
* Execute unintended operations

Command injection can be extremely serious.

---

# 12. XXE — XML External Entity

XXE occurs when an XML parser processes dangerous external entities.

Imagine an application accepts:

```xml
<user>
    <name>Mamun</name>
</user>
```

The application uses an XML parser.

If external entity processing is unnecessarily enabled, specially constructed XML can cause the parser to access external/local resources.

### Potential impact

Depending on configuration:

```text
XXE
 ↓
Local file disclosure
 ↓
SSRF
 ↓
Internal service access
```

### Common locations

* SOAP APIs
* XML APIs
* Document processors
* SVG/XML processing
* Legacy integrations

### Prevention

Disable:

* External entities
* External DTDs
* Unnecessary entity expansion

Use a hardened XML parser configuration.

---

# 13. Prototype Pollution

This is particularly important in **JavaScript/Node.js** applications.

JavaScript objects inherit properties through prototypes.

Conceptually:

```text
Object
  ↓
Prototype
  ↓
Object instances
```

If an application unsafely merges attacker-controlled objects into other objects, an attacker may influence inherited properties.

Dangerous patterns often involve:

```text
deep merge
recursive merge
object assignment
untrusted JSON
```

For example, conceptually:

```javascript
merge(target, userInput);
```

If the merge function doesn't protect special prototype-related properties, attacker-controlled input can potentially modify behavior across objects.

### Why it matters

Impact depends heavily on the application.

Possible consequences include:

* Authentication bypass
* Authorization changes
* Application logic manipulation
* Denial of service
* In some vulnerable dependency chains, code execution

### Prevention

* Use maintained libraries
* Update vulnerable dependencies
* Validate object keys
* Avoid unsafe recursive merges
* Use safer object structures such as `Object.create(null)` where appropriate
* Avoid trusting inherited properties

---

# 14. JWT Vulnerabilities

JWT = JSON Web Token.

A simplified JWT looks like:

```text
HEADER.PAYLOAD.SIGNATURE
```

Example payload:

```json
{
  "sub": "123",
  "role": "user"
}
```

### Important misconception

JWT payloads are generally **encoded, not encrypted**.

Therefore, don't put secrets in the payload unless the design specifically uses encryption.

### Common JWT problems

#### Weak signing configuration

Tokens must be validated against an appropriate expected algorithm and key.

#### Algorithm confusion

The server must not blindly trust attacker-controlled algorithm metadata.

#### Signature not verified

This is a critical implementation failure.

#### Long-lived tokens

If a token remains valid for an excessive period, compromise has a larger window.

#### Sensitive information in payload

For example:

```json
{
  "password": "...",
  "secret": "..."
}
```

Don't do this.

#### Poor key management

Signing secrets should be:

* Strong
* Random
* Protected
* Rotatable

---

# 15. API Security Issues

Modern applications are heavily API-driven.

Typical API:

```http
GET /api/users/123
```

Important API vulnerability classes include:

### BOLA

User A accesses User B's object.

```text
/api/users/123
/api/users/124
```

The important question is whether authorization is checked server-side.

### Broken Function-Level Authorization

Normal user accesses:

```text
/api/admin/users
```

### Mass assignment

Suppose the legitimate request is:

```json
{
  "name": "Mamun"
}
```

But the server blindly accepts:

```json
{
  "name": "Mamun",
  "role": "admin"
}
```

This can cause privilege problems.

### Excessive data exposure

API returns:

```json
{
  "name": "...",
  "email": "...",
  "internal_id": "...",
  "password_reset_token": "..."
}
```

even though the frontend only needs:

```json
{
  "name": "..."
}
```

### Other API issues

* Missing authentication
* Missing authorization
* Rate-limit problems
* Improper input validation
* GraphQL authorization issues
* Excessive resource consumption
* API versioning mistakes

---

# 16. Race Conditions

A race condition occurs when application behavior depends on the timing/order of simultaneous operations.

Imagine:

```text
Account balance = $100
```

Two withdrawal requests arrive almost simultaneously:

```text
Request A → withdraw $100
Request B → withdraw $100
```

If both requests check:

```text
balance >= $100
```

before either transaction updates the balance, both may succeed.

Result:

```text
Expected: $0
Actual: potentially -$100
```

### Common targets

* Money transfers
* Coupon redemption
* Gift cards
* Inventory
* One-time tokens
* Password reset flows
* Account actions
* Voting systems

### Prevention

Use:

* Database transactions
* Row locking
* Atomic operations
* Unique constraints
* Idempotency keys
* Proper state machines

---

# 17. Business Logic Vulnerabilities

These are among the most interesting bugs in bug bounty.

The application may be technically functioning correctly, but the **business rules can be abused**.

Example:

An e-commerce website says:

```text
Maximum 1 coupon per order
```

But the application doesn't actually enforce that rule server-side.

The UI may prevent:

```text
Coupon A + Coupon B
```

but the API might accept both.

### Another example

Application:

```text
Maximum withdrawal = $500/day
```

The frontend prevents another withdrawal.

But if the server doesn't maintain the daily limit correctly:

```text
Request 1 → $500
Request 2 → $500
Request 3 → $500
```

the business rule can be bypassed.

### Key mindset

Don't only ask:

> "Can I break the application?"

Ask:

> **"Can I make the application do something the business owner never intended?"**

---

# 18. Information Disclosure

Information disclosure occurs when an application reveals information that shouldn't be exposed.

Examples:

### Stack trace

```text
java.lang.NullPointerException
/home/app/src/payment/PaymentService.java
```

### API response

```json
{
  "user_id": 123,
  "internal_database_id": 987,
  "debug": true
}
```

### HTTP headers

Sometimes infrastructure information is unnecessarily exposed.

### Other examples

* Internal IP addresses
* Source-code fragments
* Database errors
* Debug information
* API keys
* Cloud credentials
* Backup files
* User information
* Internal hostnames

### Important

Not every information leak is automatically a high-severity vulnerability.

You should determine:

```text
What information?
        ↓
Who can access it?
        ↓
Is it sensitive?
        ↓
Can it be used for further impact?
```

---

# 19. HTTP Request Smuggling

This is a more advanced vulnerability.

It occurs when different components in a request chain disagree about where an HTTP request ends.

Typical architecture:

```text
Browser
   ↓
CDN / Proxy
   ↓
Load Balancer
   ↓
Web Server
```

Suppose the frontend proxy interprets a request one way while the backend interprets it differently.

The disagreement can cause:

```text
Frontend interpretation
        ≠
Backend interpretation
```

This can allow an attacker to "smuggle" part of a request into the next request.

### Common concepts

You will encounter:

```text
Content-Length
Transfer-Encoding
```

and techniques/classes such as:

```text
CL.TE
TE.CL
TE.TE
```

### Potential impact

Depending on the environment:

* Request routing manipulation
* Cache poisoning
* Authentication bypass
* Access-control bypass
* WebSocket/proxy issues
* Poisoning another user's request

This topic requires much deeper HTTP/proxy knowledge than ordinary vulnerabilities.

---

# 20. Cache Poisoning

Caching systems store responses so they can be served faster.

Architecture:

```text
User
 ↓
CDN
 ↓
Application
```

Suppose:

```text
GET /page
```

produces a response that gets cached.

If an attacker can influence something that affects the response but **isn't included in the cache key**, the attacker may cause a poisoned response to be stored.

Conceptually:

```text
Attacker request
      ↓
Application generates manipulated response
      ↓
CDN caches it
      ↓
Other users receive it
```

### Things to investigate

* Host-related behavior
* Query parameters
* Headers affecting responses
* Cache-control behavior
* Cache key construction
* Content negotiation

### Impact

Potentially:

* Stored XSS-like effects
* Wrong redirects
* Content manipulation
* User-specific content exposure

---

# 21. Web Cache Deception

This is different from cache poisoning.

The attacker attempts to make a cache treat **sensitive dynamic content** as if it were a static resource.

Imagine:

```text
https://example.com/account
```

returns private account information.

An attacker might manipulate the URL structure so that some caching layer incorrectly believes the response is a static resource.

Conceptually:

```text
/account/[cache-looking-path]
```

If the server/router still processes it as:

```text
/account
```

while the cache treats it as a cacheable static resource, sensitive information can potentially become cached.

### Difference

| Vulnerability       | Core problem                                       |
| ------------------- | -------------------------------------------------- |
| Cache poisoning     | Attacker poisons a cached response                 |
| Web cache deception | Sensitive dynamic response gets cached incorrectly |

---

# How These Vulnerabilities Connect

A very important bug-bounty skill is understanding that vulnerabilities can **chain together**.

For example:

```text
Authentication flaw
       ↓
Account takeover
       ↓
Authorization weakness
       ↓
Access another user's data
```

Another example:

```text
SSTI
 ↓
Server-side impact
 ↓
Sensitive information
```

Or:

```text
Prototype Pollution
 ↓
Application behavior manipulation
 ↓
Authorization/security impact
```

Or:

```text
HTTP Request Smuggling
        ↓
Cache poisoning
        ↓
Victim receives attacker-controlled response
```

---

# Practical Bug-Bounty Testing Mindset

When you intercept a request in Burp Suite, don't immediately start throwing payloads at it.

Ask these questions.

### 1. Authentication

```text
Who am I?
Is authentication required?
Can authentication be bypassed?
What happens after logout?
```

### 2. Authorization

```text
What object am I accessing?
Can another user access it?
What happens if roles change?
Does the server enforce permissions?
```

### 3. Session

```text
How is my session identified?
Does logout invalidate it?
Does password change invalidate old sessions?
Are cookies Secure/HttpOnly/SameSite?
```

### 4. Input

```text
Where does my input go?
Database?
Filesystem?
Template?
OS command?
XML parser?
HTML/JavaScript?
```

### 5. API

```text
Can I modify object IDs?
Can I add unexpected fields?
Can I access admin endpoints?
Does the API expose excessive data?
```

### 6. Business logic

```text
What is the intended rule?
Is that rule enforced server-side?
Can I repeat an operation?
Can I perform operations in an unexpected order?
Can two requests happen simultaneously?
```

### 7. Infrastructure

```text
Is there a CDN?
Reverse proxy?
Load balancer?
Caching?
Multiple backend servers?
```

This becomes particularly important for:

```text
HTTP Request Smuggling
Cache Poisoning
Web Cache Deception
CORS
```

---

# Suggested Learning Order

Don't try to learn all 21 simultaneously.

I'd recommend:

### Level 1 — Foundation

```text
HTTP
HTTPS
Cookies
Sessions
Authentication
Authorization
```

### Level 2 — High-value web bugs

```text
IDOR / BOLA
XSS
CSRF
SQL Injection
File Upload
Path Traversal
Open Redirect
```

### Level 3 — Server-side vulnerabilities

```text
SSRF
LFI
SSTI
Command Injection
XXE
```

### Level 4 — Modern application security

```text
JWT
API Security
CORS
Prototype Pollution
Business Logic
Race Conditions
```

### Level 5 — Advanced infrastructure

```text
HTTP Request Smuggling
Cache Poisoning
Web Cache Deception
```

---

# One Mental Model to Remember

For almost every web vulnerability, trace this flow:

```text
                    ┌──────────────┐
                    │    Browser   │
                    └──────┬───────┘
                           │
                           ▼
                    HTTP Request
                           │
                           ▼
                 ┌──────────────────┐
                 │ CDN / Proxy / WAF │
                 └────────┬─────────┘
                          │
                          ▼
                   Web Application
                          │
          ┌───────────────┼───────────────┐
          ▼               ▼               ▼
       Database        Filesystem       OS Command
          │               │               │
          └───────────────┼───────────────┘
                          ▼
                       Response
```

Then ask:

> **Where does my input go?**

If input reaches:

```text
HTML/DOM        → XSS
Database        → SQLi
Filesystem      → Path Traversal/LFI
Template engine → SSTI
OS command      → Command Injection
XML parser      → XXE
Object merge    → Prototype Pollution
Redirect        → Open Redirect
Authorization   → IDOR/BOLA
Cache           → Cache Poisoning/Deception
HTTP parser     → Request Smuggling
Business rules  → Business Logic
```

That mental model is much more valuable than memorizing hundreds of payloads.

### A particularly useful next step

For your bug-bounty learning, I would next study **Authentication → Authorization/IDOR → Session → JWT → API Security** as one connected topic. Those five areas reinforce each other and are heavily represented in modern web applications.
