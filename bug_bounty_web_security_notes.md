# Web Application Security for Bug Bounty Hunters
*A Beginner's Guide to Finding Vulnerabilities*

## Table of Contents
1. [Cross-Site Scripting (XSS)](#cross-site-scripting-xss)
2. [SQL Injection](#sql-injection)
3. [Cross-Site Request Forgery (CSRF)](#cross-site-request-forgery-csrf)
4. [Insecure Direct Object References (IDOR)](#insecure-direct-object-references-idor)
5. [Server-Side Request Forgery (SSRF)](#server-side-request-forgery-ssrf)
6. [Authentication & Authorization Flaws](#authentication--authorization-flaws)
7. [File Upload Vulnerabilities](#file-upload-vulnerabilities)
8. [Business Logic Flaws](#business-logic-flaws)
9. [Information Disclosure](#information-disclosure)
10. [Tools and Methodology](#tools-and-methodology)

---

1. Cross-Site Scripting (XSS)
CSS-XSS Exploits show us the importance of managing responsibilities in owning a website.
### What is XSS?
XSS occurs when an application includes untrusted data in a web page without proper validation or escaping, allowing attackers to execute malicious scripts in users' browsers.
XSS is the most common publicly reported security vulnerability, and part of every hacker's tkit.
### Types of XSS:

**1. Reflected XSS**
- Malicious script is reflected off the web server
- Occurs when user input is immediately returned by a web application
- Example: Search functionality that displays your search term
```
POST /search
q=<script>alert('XSS')</script>

Response shows: "Results for <script>alert('XSS')</script>"
```

**2. Stored XSS (Persistent)**
- Malicious script is stored on the server (database, files, etc.)
- Executes when other users view the stored content
- Example: Comment sections, user profiles, forum posts

**3. DOM-based XSS**
- Vulnerability exists in client-side code rather than server-side
- DOM environment is modified by malicious script
- Example: JavaScript that processes URL fragments
### Risks:
The damage that XSS could cause depends on the sensitivity of the data being handled on the site you are testing. Some things that could possibly happen are:
- **Spreading worms on social media sites** - Facebook, Twitter/X, and YouTube have all been attacked this way.
- **Session hijacking** - Malicious JavaScript may be able to send the session ID to a remote site under the hackers control, allowing the attacker to hijack a session in progress.
- **ID Theft** - If the user enters confidential info such as credit card numbers in a compromised website, these details can be unveiled using the malicious JavaScript.
- **DOS Attacks & Website Vandalism**
- **Password Theft**(Any Site)
- **Financial Fraud**(Bank Site/Investor Platform/crypto trading platforms)
### Bug Bounty Impact:
- **Critical/High severity** - Can lead to account takeover
- Common in: Comment sections, search bars, user profiles, feedback forms
- **Payout range**: $50-$5,000+ depending on impact and scope

### How to Test:
```javascript
// Basic payloads to try:
<script>alert('XSS')</script>
<img src=x onerror=alert('XSS')>
<svg onload=alert('XSS')>
javascript:alert('XSS')

// For DOM-based:
#<script>alert('XSS')</script>
```

### Prevention:
- Input validation and output encoding
- Content Security Policy (CSP)
- HTTPOnly cookies

### Protection 
To prevent stored XSS attacks, it's important to make sure that any content pulled from a database and shown on a webpage can't be used to sneak in JavaScript code.

- **Escape Dynamic Content** - Webpages are usually built using HTML templates, and dynamic content is added when the page loads. Stored XSS happens when this content isn’t handled properly. An attacker might put JavaScript into a field that gets saved in the database. Then, when someone else visits the page, that script runs in their browser. Unless your website is meant to let users write raw HTML (like a blog platform), you should always escape dynamic content. Escaping tells the browser to treat the content as plain text inside HTML tags, not as actual HTML code. Escaping usually means replacing special characters with their HTML-safe versions:
| Character | Becomes |
|-----------|---------|
| <         | &#60    |
| >         | &#62    |
| &         | &#38    |
| "         | &#34    |
| '         | &#39    |


- **Use a Safe List (Allowlist)** - If a piece of data should only have a few valid options, it’s best to limit those options in the database and only allow known good values when showing them on the page. For example, instead of letting users type their country, give them a dropdown menu to choose from.

- **Set a Content Security Policy (CSP)** - Web browsers support something called a Content Security Policy, or CSP. This lets you control where scripts can be loaded from. XSS attacks usually work by injecting scripts into a page or loading them from a bad website. You can stop this by adding a CSP header to your site’s responses:

- Example of setting a CSP header to your site's responses
'''html
Content-Security-Policy: script-src 'self' https://apis.google.com
'''
- This tells the browser to only run scripts from your site and trusted sources like Google APIs. You can also add this in your HTML:

// Example
```html
<meta http-equiv="Content-Security-Policy" content="script-src 'self' https://apis.google.com">
```
This is a strong way to protect users, but it might take some effort to get your site ready. Inline scripts (scripts written directly in HTML) are discouraged in modern web development, but they’re still common in older sites.
'''html
 Reporting-Endpoints: csp-endpoint="https://example.com/csp-reports"
 Content-Security-Policy-Report-Only: script-src 'self'; report-to csp-endpoint
'''
To slowly move away from inline scripts, you can use CSP reports. These let you test your policy without blocking anything yet:


This way, you’ll get alerts about any violations before fully enforcing the policy.

- **Sanitize HTML**
If your site needs to store and show raw HTML (like for rich text posts), you should use an HTML sanitizer. This tool cleans up the HTML to make sure users can’t sneak in harmful scripts.

2. SQL Injection

### What is SQL Injection?
Occurs when an attacker can insert malicious SQL code into application queries, potentially accessing or modifying database data. It is known to be one of the most common types of attacks on the internet and there are many different types of sql injection.
### Targets:
Database
### Types:

**1. Classic SQL Injection**
```sql
-- Vulnerable query:
SELECT * FROM users WHERE username = '$username' AND password = '$password'

-- Malicious input:
username: admin'--
password: anything

-- Resulting query:
SELECT * FROM users WHERE username = 'admin'--' AND password = 'anything'
```

**2. Blind SQL Injection**
- No direct output from database
- Attacker infers information from application behavior
- Time-based: `'; WAITFOR DELAY '00:00:05'--`
- Boolean-based: True/false responses

**3. Union-based SQL Injection**
```sql
' UNION SELECT username, password FROM users--
```
### Risks "Whats the worst thing that could happen?"
Complex attacks allow the malicious user to run arbitrary statements on that database. Hackers use injection to:
- **Extract sensitive info** - SSN, Credit Card Numbers, Addresses, Family Members.
- **Enumerate the auth details of users registered** - Allows for the harvesting and reuse of stolen emails for phishing campaigns.
- **Corrupt data** - Delete, drop, and make the website unusable.
- **Inject layers of malicious code** - Executed when users visit the site.

Note: Don't let organizations lie to you and make empty promises of secure data. Your data is never guarunteed to be safe.
  SQL Injection has a reputation of having been used on major companies like Yahoo and Sony.

### Bug Bounty Impact:
- **Critical severity** - Can lead to full database compromise
- Common in: Login forms, search functions, API endpoints
- **Payout range**: $500-$15,000+ for critical instances

### Basic Testing:
```sql
# Test for errors:
'
''
`
``
,
"
""
/
//
\
\\
and 1=1
and 1=2
' and 1=1--
' and 1=2--
```

### Tools:
- SQLMap (automated)
- Burp Suite
- Manual testing with Burp Repeater

### How do we defend ourselves?:
 - Parameterized Statements
 - Object Relational Mapping
 - Escaping Inputs
 - Sanitizing Inputs

---

## Command Execution Attacks

### What is Command Execution?
Occurs when an attacker can execute system commands through a vulnerable application. This includes OS command injection, remote code execution (RCE), and server-side template injection.
### Targets:
Host System
### Types:

**1. OS Command Injection**
```bash
# Basic payload examples:
ping 127.0.0.1 ; ls    # Command chaining
ping $(cat /etc/passwd)  # Command substitution
ping `whoami`          # Backtick execution
ping || dir           # Windows command injection
```

**2. Remote Code Execution (RCE)**
```php
# PHP example:
?page=php://input    # PHP wrapper
?page=expect://id    # PHP expect
?eval=system('ls')   # Direct code execution

# Java example:
${T(java.lang.Runtime).getRuntime().exec('calc.exe')}
```

**3. Server-Side Template Injection**
```python
# Template injection examples:
${7*7}               # Basic test
#{7*7}               # Alternative syntax
${class.getClassLoader()}  # Java class access
{{7*7}}              # Twig/Jinja2 syntax
```

### Risks:
- Complete system compromise
- Data theft or destruction
- Malware installation
- Lateral movement in networks
- Cryptocurrency mining
- Backdoor installation
- Service disruption
- Infrastructure damage
- Ransomware deployment
- Compliance violations

### Bug Bounty Impact:
- **Critical severity** - Often leads to server compromise
- Common in: File processors, admin panels, template engines
- **Payout range**: $1,000-$25,000+

### Prevention:
1. Never pass user input to system commands
2. Use allowlists for permitted commands
3. Implement proper input validation
4. Avoid dangerous PHP functions (eval, system, exec)
5. Use secure template configurations

### Testing Tools:
- commix - Command injection exploiter
- tplmap - Template injection scanner
- Custom payloads with Burp Suite

---

3. Cross-Site Request Forgery (CSRF)

### What is CSRF?
Forces authenticated users to execute unwanted actions on a web application where they're authenticated.

### How it Works:
1. Victim is logged into vulnerable site
2. Attacker tricks victim into visiting malicious page
3. Malicious page sends request to vulnerable site
4. Request executes with victim's credentials
### Targets:
Active Sessions
### Example Attack:
```html
<!-- Malicious HTML that transfers money -->
<form action="https://bank.com/transfer" method="POST">
  <input type="hidden" name="to" value="attacker_account">
  <input type="hidden" name="amount" value="1000">
</form>
<script>document.forms[0].submit();</script>
```

### Bug Bounty Impact:
- **Medium to High severity**
- Common in: State-changing operations (password change, money transfer, account settings)
- **Payout range**: $100-$2,000+

### How to Test:
1. Find state-changing requests (POST, PUT, DELETE)
2. Remove CSRF tokens if present
3. Create proof-of-concept HTML page
4. Test if action executes without proper verification

### CSRF PoC Template:
```html
<html>
<body>
  <form action="https://target.com/change-email" method="POST">
    <input type="hidden" name="email" value="attacker@evil.com">
    <input type="submit" value="Click me">
  </form>
</body>
</html>
```

---

4. Insecure Direct Object References (IDOR)

### What is IDOR?
Occurs when an application provides direct access to objects based on user-supplied input, allowing unauthorized access to data.

### Common Examples:
```
# Accessing other users' data:
https://app.com/user/profile?id=123  # Your profile
https://app.com/user/profile?id=124  # Someone else's profile

# Downloading files:
https://app.com/download/invoice/1001  # Your invoice
https://app.com/download/invoice/1002  # Someone else's invoice
```
### Targets:
  1. **Sensitive Data**
  2. **User Accounts**
  3. **Horizontal Privelage Escalation**
  4. **Vertical Privelage Escalation**
### Types:
- **Numeric IDs**: 1, 2, 3, 4...
- **GUIDs**: But sometimes predictable or enumerable
- **Encoded IDs**: Base64, etc. (try decoding)
### Examples:
- **Direct Reference to Database Objects**: An attacker can modify the customer number in a URL to access other customers' records, bypassing access controls 1.
- **Static Files**: An attacker can change the filename in a URL to retrieve sensitive files stored on the server
### Bug Bounty Impact:
- **Medium to High severity**
- Very common vulnerability
- **Payout range**: $200-$3,000+

### How to Test:
1. Find URLs with ID parameters
2. Note your legitimate ID
3. Try accessing other users' data by changing ID
4. Test with Burp Intruder for enumeration

### Tools:
- Burp Suite Intruder
- Custom scripts for automation
- Browser developer tools

---

5. Server-Side Request Forgery (SSRF)

### What is SSRF?
Attacker can abuse server functionality to read or update internal resources, making requests on behalf of the server.

### Impact:
- Access internal services
- Read local files
- Port scanning internal network
- Cloud metadata access (AWS, GCP, Azure)

### Common Vulnerable Features:
- Image/document processing from URLs
- Webhooks
- URL preview functionality
- API integrations

### Bug Bounty Impact:
- **High to Critical severity**
- **Payout range**: $500-$10,000+

### Basic Payloads:
```
# Internal network scanning:
http://127.0.0.1:80
http://127.0.0.1:22
http://127.0.0.1:3306
http://192.168.1.1
http://10.0.0.1

# Cloud metadata (AWS):
http://169.254.169.254/latest/meta-data/
http://169.254.169.254/latest/meta-data/iam/security-credentials/

# Local file access:
file:///etc/passwd
file:///etc/hosts
file://C:\Windows\System32\drivers\etc\hosts
```

### Bypass Techniques:
```
# IP encoding:
http://2130706433/  # 127.0.0.1 in decimal
http://0x7f000001/  # 127.0.0.1 in hex
http://127.1/       # Short form

# URL schemes:
gopher://
ftp://
dict://
ldap://
```

---

## Authentication & Authorization Flaws

### Common Issues:

**1. Broken Authentication**
- Weak password policies
- Session management flaws
- Credential stuffing vulnerabilities

**2. Broken Access Control**
- Privilege escalation
- Missing function-level access control
- Insecure direct object references

**3. JWT Vulnerabilities**
```json
# None algorithm attack:
{
  "alg": "none",
  "typ": "JWT"
}

# Weak secret brute force
# Algorithm confusion (RS256 to HS256)
```

### Bug Bounty Impact:
- **High to Critical severity**
- **Payout range**: $300-$8,000+

### Testing Approach:
1. Test user registration/login flows
2. Check session management
3. Test privilege escalation
4. Analyze JWT tokens
5. Test password reset functionality

---

## File Upload Vulnerabilities

### What to Test:
1. **File type restrictions bypass**
2. **Malicious file upload**
3. **Path traversal in uploads**
4. **Server-side code execution**

### Common Bypasses:
```
# Extension bypasses:
shell.php.jpg
shell.php%00.jpg
shell.php;.jpg
shell.pHp

# MIME type spoofing:
Content-Type: image/jpeg  # For a PHP file

# Magic bytes:
Add GIF89a to beginning of PHP file
```

### Risks:
- Remote code execution through malicious files
- Web shell upload and system compromise
- Server-side file inclusion
- Denial of service through large files
- Overwriting critical system files
- Cross-site scripting via SVG uploads
- Malware distribution
- Content spoofing

### Protection:
1. File Validation
   - Validate file extensions
   - Check MIME types
   - Verify file content matches type
   - Implement file size limits

2. Storage Security
   - Use separate domains for uploaded content
   - Randomize filenames
   - Set proper permissions
   - Scan files for malware

3. Access Controls
   - Implement authentication for uploads
   - Restrict upload directory permissions
   - Use secure file paths
   - Prevent directory traversal

## Business Logic Flaws

### What are Business Logic Flaws?
Vulnerabilities in the application's workflow that allow attackers to misuse legitimate functionality.

### Common Examples:
- **Race conditions**: Multiple simultaneous requests
- **Price manipulation**: Negative quantities, currency issues
- **Workflow bypasses**: Skipping payment steps
- **Rate limiting bypasses**

### Bug Bounty Impact:
- **Medium to Critical severity**
- Often unique and high-impact
- **Payout range**: $300-$15,000+

### Testing Approach:
1. Understand the application's business logic
2. Think like an attacker: "How can I abuse this?"
3. Test edge cases and unexpected inputs
4. Look for race conditions
5. Test multi-step processes

### Risks:
- Financial losses through price manipulation
- Unauthorized access to features
- Account takeover through workflow bypass
- Service abuse through rate limit bypass
- Data leakage through logic errors
- Inventory manipulation
- Loyalty points/rewards abuse
- Transaction tampering

### Protection:
1. Input Validation
   - Validate all business rules
   - Implement server-side checks
   - Verify transaction sequences
   - Validate numerical inputs

2. Process Controls
   - Implement proper state management
   - Add transaction limits
   - Monitor for abuse patterns
   - Enforce workflow sequences

3. Rate Limiting
   - Implement proper rate limits
   - Use robust session management
   - Monitor for suspicious activity
   - Track user behavior patterns

## Information Disclosure

### Types:
- **Debug information**: Stack traces, error messages
- **Source code exposure**: .git folders, backup files
- **Sensitive data in responses**: PII, internal system info
- **Directory listings**
- **Configuration files**

### Common Findings:
```
# Common sensitive files:
/.git/
/backup/
/.env
/config.php
/phpinfo.php
/.DS_Store
/robots.txt
/sitemap.xml

# Error-based info disclosure:
SQL errors revealing database structure
PHP errors showing file paths
Stack traces in development mode
```

### Risks:
- Exposure of sensitive user data
- System configuration leakage
- Source code disclosure
- Internal IP/hostname exposure
- Database structure revelation
- Authentication bypass hints
- Development data exposure
- API key/credential leakage

### Protection:
1. Error Handling
   - Custom error pages
   - Generic error messages
   - Log sensitive errors server-side
   - Remove debug information

2. Security Headers
   - Implement proper security headers
   - Remove version information
   - Disable directory listing
   - Control CORS policies

3. Configuration Management
   - Secure version control
   - Remove backup files
   - Protect configuration files
   - Use environment variables

---

## Tools and Methodology

### Essential Tools:

**1. Burp Suite**
- Proxy, repeater, intruder, scanner
- Essential for manual testing

**2. OWASP ZAP**
- Free alternative to Burp
- Good automated scanning

**3. Reconnaissance Tools**
```bash
# Subdomain enumeration:
Subfinder, Amass, Assetfinder

# Directory brute forcing:
Gobuster, Dirb, Dirsearch

# Port scanning:
Nmap, Masscan
```

**4. Specialized Tools**
- **SQLMap**: SQL injection testing
- **XSStrike**: XSS detection
- **Nuclei**: Template-based vulnerability scanner
- **ffuf**: Fast web fuzzer

### Bug Bounty Methodology:

**1. Reconnaissance (30% of time)**
- Subdomain enumeration
- Technology identification
- Directory/file discovery
- Parameter discovery

**2. Vulnerability Assessment (50% of time)**
- Automated scanning
- Manual testing of key functionalities
- Business logic analysis

**3. Exploitation & Documentation (20% of time)**
- Proof of concept development
- Impact assessment
- Report writing

### Report Writing Tips:
1. **Clear title**: "SQL Injection in login form leads to full database access"
2. **Severity assessment**: Use CVSS or platform guidelines
3. **Step-by-step reproduction**
4. **Proof of concept**: Screenshots, payloads, responses
5. **Impact explanation**: Business impact, not just technical
6. **Remediation suggestions**
7. **Example Reports**: It's a great idea to look at examples of bug bounty hunt reports to understand how to create one effectively. Reviewing these examples can provide valuable insights into the structure, content, and level of detail required in a report.
**Benefits of Reviewing Examples**:
  1. Understanding Structure: You can learn how to organize your findings, including sections like summary, steps to reproduce, impact, and recommendations.
  2. Learning Best Practices: Examples often showcase best practices in writing clear and concise reports, which is crucial for communicating your findings effectively.
  3. Identifying Common Vulnerabilities: By seeing what others have reported, you can get a sense of common vulnerabilities and how they are described.
  4. Improving Your Skills: Analyzing well-written reports can help you improve your own reporting skills, making your submissions more likely to be accepted and rewarded.
**Where to Find Examples**:
  1. Bug Bounty Platforms: Websites like HackerOne and Bugcrowd often have public disclosure sections where you can read reports submitted by other hunters.
  2. Community Forums: Online communities and forums dedicated to bug bounty hunting may share examples and templates.
  3. Educational Resources: Some cybersecurity courses and tutorials include sample reports to help learners understand the reporting process.

### Getting Started Tips:

**1. Start with Public Programs**
- Lower competition on some programs
- Good for learning

**2. Focus on Learning**
- Quality over quantity
- Understand the vulnerability deeply

**3. Practice Platforms**
- PortSwigger Web Security Academy
- DVWA (Damn Vulnerable Web Application)
- bWAPP
- HackTheBox
- TryHackMe

**4. Stay Updated**
- Follow security researchers on Twitter
- Read bug bounty writeups
- Subscribe to security blogs

**5. Common Beginner Mistakes to Avoid**
- Don't scan aggressively (rate limiting)
- Always read the scope carefully
- Don't submit duplicates
- Test in staging/development environments when possible
- Don't impact production systems

---

## Final Notes for Bug Bounty Success

### Key Statistics:
- **Average bug bounty payout**: $500-2,000
- **Time to find first bug**: 2-6 months for beginners
- **Most common vulnerabilities**: XSS (23%), IDOR (18%), Info Disclosure (16%)

### Building Your Reputation:
1. Start with smaller programs
2. Write detailed, high-quality reports
3. Be patient and persistent
4. Learn from rejected reports
5. Build relationships with program teams

Remember: Bug bounty hunting requires patience, continuous learning, and ethical behavior. Always follow the program's rules and responsible disclosure guidelines.

---

*Last updated: June 2025*
*This guide covers fundamental concepts - continue learning and practicing to become proficient.*

