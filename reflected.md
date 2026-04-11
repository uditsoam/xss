# 🟡 Reflected XSS (Non-Persistent XSS) – Complete Notes

---

## 📌 Introduction

Reflected XSS is a type of **Non-Persistent XSS** where:

* Input is processed by the backend server
* Returned in the response
* Not stored in the database

The attack exists only for a single request and disappears after refresh.

---

## 📌 What is Reflected XSS?

Reflected XSS occurs when:

* User input is sent to the server
* Server reflects the same input in the response
* Input is not sanitized
* JavaScript executes in the browser

---

## 📌 Key Characteristics

* Not stored in database
* Executes only once
* Disappears after refresh
* Targets specific user
* Requires user interaction (clicking link)

---

## 📌 Practical Example

### Scenario (To-Do App)

* User enters a task
* Application shows error message including user input

If the application:

* Reflects input in response
* Does not sanitize input

Then it is vulnerable to Reflected XSS.

---

## 📌 XSS Testing Payloads

### Basic Payload

```html
<script>alert(window.origin)</script>
```

### Simple Alert Payload

```html
<script>alert(1)</script>
```

### Print Payload

```html
<script>print()</script>
```

### Plaintext Payload

```html
<plaintext>
```

---

## 📌 Where Payload is Used

### Common Injection Points

* Search bars
* Error messages
* URL parameters (`?q=`, `?task=`)
* Login forms
* Filters and query inputs

---

## 📌 Execution Process

### Step-by-Step

1. Inject payload in input field
2. Submit request
3. Server reflects input in response
4. Browser executes script

---

## 📌 Example URL Attack

```bash
http://target.com/?task=<script>alert(window.origin)</script>
```

### Encoded Payload

```bash
http://target.com/?task=%3Cscript%3Ealert(1)%3C/script%3E
```

Used to bypass filters.

---

## 📌 Behavior Observation

### Output Example

```html
Task '' could not be added.
```

### Reason

* Script executed
* Not displayed as text

---

## 📌 Page Source Verification

Check using:

```bash
CTRL + U
```

### Example Source

```html
Task '<script>alert(window.origin)</script>' could not be added.
```

### Conclusion

* Input is reflected in HTML
* No sanitization applied

---

## 📌 Non-Persistent Nature

### Test

* Refresh the page

### Result

* Alert does not appear again
* Payload disappears

This confirms Reflected XSS.

---

## 📌 How to Exploit Reflected XSS

### Step 1: Check Request Type

Open DevTools → Network tab

If request is:

```bash
GET
```

---

### Step 2: Create Malicious URL

```bash
http://target.com/?q=<script>alert(1)</script>
```

---

### Step 3: Send to Victim

* Email
* WhatsApp
* Phishing link

---

### Step 4: Victim Clicks

* Script executes in browser

---

## 📌 Why Reflected XSS Needs Social Engineering

* Not stored
* Works only when link is opened
* Requires user interaction

---

## 📌 Impact of Reflected XSS

* Cookie stealing
* Account takeover
* Phishing attacks
* Redirecting users
* Executing malicious actions

---

## 📌 Key Indicators

| Behavior                 | Meaning              |
| ------------------------ | -------------------- |
| Runs once                | Reflected XSS        |
| Disappears after refresh | Non-Persistent       |
| Requires link click      | User-targeted attack |

---

## 📌 Difference from Stored XSS

| Feature     | Stored XSS   | Reflected XSS |
| ----------- | ------------ | ------------- |
| Storage     | Database     | Not stored    |
| Execution   | Every user   | Specific user |
| Persistence | Yes          | No            |
| Interaction | Not required | Required      |

---

## 📌 Real Response Example (Vulnerable Output)

```html
<div></div>
<ul class="list-unstyled" id="todo">
  <div style="padding-left:25px">
    Task '<script>alert(window.origin)</script>' could not be added.
  </div>
</ul>
```

### Explanation

* User input is directly embedded in HTML
* Script is not sanitized
* Browser executes JavaScript
* Confirms Reflected XSS vulnerability

---

## 🧠 Key Takeaways

* Reflected XSS is temporary and request-based
* Payload is reflected in response
* Requires user interaction
* Easily exploitable via malicious links
* Always verify reflection + execution

---
