## 1. What is DOM XSS?

DOM-Based XSS is a type of Non-Persistent XSS where:

The attack is executed entirely in the browser (client-side)
The input never reaches the backend server
JavaScript modifies the page using the DOM (Document Object Model)

###  Simple definition:
DOM XSS occurs when JavaScript takes user input and writes it into the page without proper sanitization.


## 2. How DOM XSS Works (Flow)
 Step-by-step:
User input is placed in the URL (usually after #)
JavaScript reads this input from the URL
JavaScript writes it into the page (DOM)
Browser executes it


### 3. Example (To-Do App)
Input:
  test
URL:
   http://site.com/#task=test
Output:
   Next Task: test


## 4. Key Observation (Very Important)
Open DevTools → Network tab
Add input again
You will see NO HTTP request

This means:
Server is NOT involved
Everything happens in the browser


## 5. URL Fragment (#)
 http://site.com/#task=test
 #task=test is called a URL fragment
It is processed only in the browser
It is NOT sent to the server

 This is a strong indicator of DOM XSS


## 6. Why Input is Not in Page Source

 Press CTRL + U (View Page Source)
You will NOT find your input
 Reason:
JavaScript updates the page AFTER load
So input is not in original HTML
 Use:
Inspect Element (CTRL + SHIFT + C) to see it


## 7. Source & Sink Concept (Very Important )
### Source (Where input comes from)
var pos = document.URL.indexOf("task=");
var task = document.URL.substring(pos + 5);
 Input is taken from:
URL
 This is called SOURCE
### Sink (Where input is written)
document.getElementById("todo").innerHTML =
"<b>Next Task:</b> " + decodeURIComponent(task);
 Input is written into:
DOM
 This is called SINK

## 8. Why DOM XSS Happens
Attacker controls the input (Source)
Input is written to DOM (Sink)
No sanitization is applied

 Result: XSS vulnerability

## 9. Dangerous Sink Functions
 Common vulnerable functions:
innerHTML
outerHTML
document.write()
 jQuery functions:
append()
after()
add()
 If these are used without sanitization → XSS possible


## 10. Why <script> Does Not Work
<script>alert(1)</script>
 Usually does NOT execute in DOM XSS ❌
 Reason:

innerHTML blocks <script> tags

## 11. Working DOM XSS Payload
 Payload:
<img src="" onerror=alert(window.origin)>


## 12. How This Payload Works
<img src="">
Image fails to load
onerror=alert(window.origin)
Executes JavaScript on error
 Result:
Alert popup appears


## 13. Full Attack URL
http://site.com/#task=<img src="" onerror=alert(window.origin)>
 When victim opens:
JavaScript executes


## 14. How to Detect DOM XSS
 Steps:
Check URL for # parameter
Open DevTools → Network tab
No request → client-side
Inspect JavaScript code
### Look for:
innerHTML
document.write()
Inject payload:
<img src=x onerror=alert(1)>





