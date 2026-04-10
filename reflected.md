# 🟡 Reflected XSS (Non-Persistent XSS) – Complete Notes

## 📌 Introduction
Reflected XSS is a type of **Non-Persistent XSS** where:
- Input is processed by the backend server  
- Returned in the response  
- Not stored in the database  

👉 The attack exists only for a single request and disappears after refresh.

---

## 📌 What is Reflected XSS?
Reflected XSS occurs when:
- User input is sent to the server  
- Server reflects the same input in the response  
- Input is NOT sanitized  
- JavaScript executes in the browser  

---

## 📌 Key Characteristics
- Not stored in database  
- Executes only once  
- Disappears after refresh  
- Targets specific user  
- Requires user interaction (clicking link)  

---

## 📌 Practical Example

### Scenario (To-Do App)
- User enters a task  
- Application shows error message including user input  


#### If application:
 Reflects input in response
 Does not sanitize input
👉 Then it is vulnerable to Reflected XSS


### 📌 XSS Testing Payloads
🔥 Basic Payload
<script>alert(window.origin)</script>

👉 Use:
Input fields

URL parameters
🔹 Simple Alert Payload
<script>alert(1)</script>
🔹 Print Payload (less blocked)
<script>print()</script>
🔹 Plaintext Payload
<plaintext>


### 📌 Where Payload is Used
🔍 Common Injection Points
Search bars
Error messages
URL parameters (?q=, ?task=)
Login forms
Filters and query inputs

### 📌 Execution Process
Step-by-Step:
Inject payload in input field
Click submit / send request
Server reflects input in response
Browser executes script

### 📌 Example URL Attack
http://target.com/?task=<script>alert(window.origin)</script>

#### 👉 When victim opens link:

Script executes
📌 Encoded Payload (Important)
http://target.com/?task=%3Cscript%3Ealert(1)%3C/script%3E

👉 Used to bypass filters

📌 Behavior Observation
Output:
Task '' could not be added.

### 👉 Reason:

Script executed
Not displayed as text
📌 Page Source Verification
Check using:
Right click → View Page Source
OR press CTRL + U
Example:
Task '<script>alert(window.origin)</script>' could not be added.

👉 Confirms:

Input reflected in HTML
No sanitization
📌 Non-Persistent Nature
Test:
Refresh page
Result:
Alert does NOT appear again
Payload disappears

👉 Confirms Reflected XSS

📌 How to Exploit Reflected XSS
Step 1: Check Request Type
Open DevTools → Network tab

👉 If:

GET request
Step 2: Create Malicious URL
http://target.com/?q=<script>alert(1)</script>
Step 3: Send to Victim
Email
WhatsApp
Phishing
Step 4: Victim Clicks



## 👉 Script executes in browser

### 📌 Why Reflected XSS Needs Social Engineering
Not stored
Only works when link is opened
Requires victim interaction
### 📌 Impact of Reflected XSS
Cookie stealing
Account takeover
Phishing attacks
Redirecting users
Executing malicious actions
### 📌 Key Indicators
Behavior	Meaning
Runs once	Reflected XSS
Disappears after refresh	Non-Persistent
Requires link click	User-targeted attack
### 📌 Difference from Stored XSS
Feature	Stored XSS	Reflected XSS
Storage	Database	Not stored
Execution	Every user	Specific user
Persistence	Yes	No
Interaction	Not required	Required





<div></div><ul class="list-unstyled" id="todo"><div style="padding-left:25px">Task '<script>alert(window.origin)</script>' could not be added.</div></ul>
