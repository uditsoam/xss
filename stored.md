# 🔴 Stored XSS (Persistent XSS) – Complete Notes

## 📌 Introduction
Before discovering and exploiting XSS vulnerabilities, it is important to understand the different types of XSS. This helps in choosing the correct attack technique.
Stored XSS is the **most critical type of XSS vulnerability**.

## 📌 What is Stored XSS?
Stored XSS (Persistent XSS) occurs when:

- Malicious input is **stored in the backend database**
- It is **retrieved when the page loads**
- The script **executes in the user's browser**

👉 This makes the attack **persistent**

## 📌  Why Stored XSS is Dangerous
- Affects **all users** visiting the page  
- No need to trick users (no link required)  
- Executes automatically on page load  
- Hard to remove (stored in database)  
- Can impact a large number of users  

---

## Vulnerability Condition

If the application:

Does NOT sanitize user input
Directly renders input into HTML

👉 Then it is vulnerable to XSS


XSS Testing Payload
Basic Payload
        <script>alert(window.origin)</script>


## Page Source Verification
Steps:
Right click → View Page Source
OR press CTRL + U
Example Output:
<div></div>
<ul class="list-unstyled" id="todo">
<ul>
<script>alert(window.origin)</script>
</ul>
</ul>

👉 This confirms:

Payload is injected into HTML
No input sanitization

## How to Confirm Stored XSS
✔ Step 1: Inject payload
Enter script in input field
✔ Step 2: Check execution
Alert popup appears
✔ Step 3: Refresh page
If alert appears again → Stored XSS confirmed

👉 Because:

Payload is coming from database, not just request



## Key Indicator of Stored XSS
Behavior	Meaning
Runs once	Reflected XSS
Runs after refresh	Stored XSS
