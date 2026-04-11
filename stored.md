# 🔴 Stored XSS (Persistent XSS) – Complete Notes

---

## 📌 Introduction

Before discovering and exploiting XSS vulnerabilities, it is important to understand the different types of XSS. This helps in choosing the correct attack technique.

Stored XSS is the **most critical type of XSS vulnerability**.

---

## 📌 What is Stored XSS?

Stored XSS (Persistent XSS) occurs when:

* Malicious input is **stored in the backend database**
* It is **retrieved when the page loads**
* The script **executes in the user's browser**

This makes the attack **persistent**.

---

## 📌 Why Stored XSS is Dangerous

* Affects **all users** visiting the page
* No need to trick users (no link required)
* Executes automatically on page load
* Hard to remove (stored in database)
* Can impact a large number of users

---

## 📌 Vulnerability Condition

If the application:

* Does not sanitize user input
* Directly renders input into HTML

Then it is vulnerable to XSS.

---

## 📌 XSS Testing Payload

### Basic Payload

```html
<script>alert(window.origin)</script>
```

---

## 📌 Page Source Verification

### Steps

```bash
Right click → View Page Source
CTRL + U
```

---

### Example Output

```html
<div></div>
<ul class="list-unstyled" id="todo">
  <ul>
    <script>alert(window.origin)</script>
  </ul>
</ul>
```

---

### Conclusion

* Payload is injected into HTML
* No input sanitization

---

## 📌 How to Confirm Stored XSS

### Step 1: Inject Payload

* Enter script in input field

---

### Step 2: Check Execution

* Alert popup appears

---

### Step 3: Refresh Page

* If alert appears again → Stored XSS confirmed

---

### Reason

* Payload is loaded from database
* Not dependent on a single request

---

## 📌 Key Indicator of Stored XSS

| Behavior           | Meaning       |
| ------------------ | ------------- |
| Runs once          | Reflected XSS |
| Runs after refresh | Stored XSS    |

---

## 🧠 Key Takeaways

* Stored XSS is persistent and database-based
* Executes automatically for all users
* Does not require user interaction
* Most impactful XSS type
* Always verify persistence after refresh

---
