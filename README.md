# Knowledge-of-Common-Web-Vulnerabilities
# Knowledge of Common Web Vulnerabilities

If you're preparing for **web security / bug bounty**, this means being able to recognize common vulnerability classes, understand **why they happen**, understand their impact, and know how to validate them safely in an authorized environment.

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

**That mindset is the real meaning of "knowledge of common web vulnerabilities."** It is much more valuable than memorizing a list of payloads.
