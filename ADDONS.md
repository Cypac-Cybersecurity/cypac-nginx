# README for Custom Nginx Container with PHPMailer and Headless Chromium

## Overview

This container is based on `ghcr.io/linuxserver/baseimage-alpine-nginx:3.21` with additional support for:

- **PHPMailer v6.9.3** for sending emails (installed at `/opt/phpmailer`)
- **Headless Chromium** for HTML to PDF conversion (installed and available as `chromium-browser` command)

## Using PHPMailer

PHPMailer is installed at `/opt/phpmailer`. To use it in your PHP scripts:

```php
<?php
// Load PHPMailer classes
require '/opt/phpmailer/src/PHPMailer.php';
require '/opt/phpmailer/src/SMTP.php';
require '/opt/phpmailer/src/Exception.php';

use PHPMailer\PHPMailer\PHPMailer;
use PHPMailer\PHPMailer\Exception;

$mail = new PHPMailer(true);

try {
    // Server settings
    $mail->isSMTP();
    $mail->Host = 'smtp.example.com';
    $mail->SMTPAuth = true;
    $mail->Username = 'yourusername@example.com';
    $mail->Password = 'yourpassword';
    $mail->SMTPSecure = PHPMailer::ENCRYPTION_STARTTLS;
    $mail->Port = 587;

    // Recipients
    $mail->setFrom('from@example.com', 'Mailer');
    $mail->addAddress('client@example.com', 'Client Name');

    // Content
    $mail->isHTML(true);
    $mail->Subject = 'Here is the subject';
    $mail->Body    = 'This is the HTML message body <b>in bold!</b>';

    $mail->send();
    echo 'Message has been sent';
} catch (Exception $e) {
    echo "Message could not be sent. Mailer Error: {$mail->ErrorInfo}";
}
?>
```

## Using Headless Chromium for PDF Generation

Headless Chromium is installed and can be used to generate PDFs from HTML files. Example:

```bash
chromium-browser --headless --disable-gpu --print-to-pdf=/tmp/output.pdf http://example.com
```

This command converts `http://example.com` into `output.pdf` while preserving print styles.

You can also invoke it from PHP:

```php
<?php
$url = 'http://example.com';
$outputPdf = '/var/www/html/output.pdf';

$command = "chromium-browser --headless --disable-gpu --no-sandbox --print-to-pdf=$outputPdf $url";
exec($command, $output, $returnCode);

if ($returnCode === 0) {
    echo "PDF successfully created at $outputPdf";
} else {
    echo "Failed to generate PDF";
}
?>
```

## Container Usage

To run the container:

```bash
docker run -d   --name=nginx   -e PUID=1000   -e PGID=1000   -e TZ=Etc/UTC   -p 80:80   -p 443:443   -v /path/to/nginx/config:/config   --restart unless-stopped   ghcr.io/cypac-cybersecurity/cypac-nginx:latest
```

## Updating the Container

To pull updates:

```bash
docker pull ghcr.io/cypac-cybersecurity/cypac-nginx:latest
docker stop nginx
docker rm nginx
docker run -d --name=nginx ... (same parameters as above)
```

For further details, refer to the documentation.
