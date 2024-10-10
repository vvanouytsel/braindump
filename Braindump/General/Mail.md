#linux #postfix #smtp

## Sendmail

You can send an email via `sendmail`.

```bash
$ echo -e "Subject: Subject\nFrom: myserver@company.com\nTo: someone@company.com\n\nTest" | sendmail -t
```

## Telnet

It is also possible to directly connect to an SMTP server and send an email via telnet.

* Set up the connection

```bash
# Make sure to add the same newlines as shown below.
# For every line you type, you will receive some feedback in your telnet prompt.
# This feedback is redacted below, to keep it simple to read.
$ telnet mailserver 25
ehlo myserver.company.com
MAIL FROM: sender@company.com
RCPT TO: recipient@company.com
DATA 
From: sender@company.com
To: recipient@company.com
Subject: test

This is the body of my email. When done, make sure to use the 'point'.
.
```
