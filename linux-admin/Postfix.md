# Postfix
Using Postfix to send messages to local Linux users is a straightforward process. Postfix, a mail transfer agent (MTA), can handle sending, receiving, and forwarding emails. When you send an email to a local user, Postfix delivers the message to the user's mailbox file in the /var/mail/ directory (or /var/spool/mail/ depending on the distribution), which can be read by mail clients like mail or mutt. 

### mailq 
Purpose: mailq is a command-line utility used to display the mail queue on systems that run a mail server, such as Sendmail, Postfix, Exim, etc. The mail queue contains messages that have been processed by the mail server but have not yet been delivered. This can include emails waiting to be sent to remote servers or messages that have been deferred due to temporary issues, such as network problems or recipient server issues.
    
Functionality: When you run mailq, it lists all the emails currently in the queue, showing details like the queue ID, sender, size, time the email was queued, and the recipient addresses. It's particularly useful for administrators to monitor and troubleshoot email delivery issues.
    
Typical Output: The output includes the queue ID, message size, the time when the message was queued, sender, and the intended recipients. It may also show the reason why a message is still in the queue if it hasn't been delivered.
### mail
Purpose: The mail command (part of the mailx or mailutils package depending on the distribution) is a simple, interactive utility for reading and sending emails from the command line. It can be used to compose, send, receive, and read emails from the system's local mailboxes or through network protocols like IMAP and SMTP when properly configured.
    
Functionality: With mail, you can send emails to local or remote users, read your emails in a text-based interface, reply to emails, and perform basic email management tasks. It is often used for reading system-generated emails (like cron job reports) in local user mailboxes, or for sending out quick emails from scripts or the command line.
    
Typical Usage: You can use mail to read your mailbox by simply typing mail with no arguments. To send an email, you might use a command like echo "This is the message body" | mail -s "Subject" recipient@example.com, where -s specifies the subject.

### Send local messages to Linux user
Ensure Postfix is Installed
```
postconf -d | grep mail_version
```
For local delivery, Postfix needs little to no configuration. However, ensure that your /etc/postfix/main.cf (Postfix's main configuration file) contains the following line to enable local delivery:
```
mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain
```
This setting tells Postfix to deliver emails locally if they are addressed to the hostname, localhost, or your domain.

After making changes to the configuration, restart Postfix to apply the changes:
```
sudo systemctl restart postfix
```

Send a Message to a Local User
```
echo "This is the test" | mail -s "Hello my dear friend" frodo@gitlab-server-01.localdomain
```

Reading the Message

The local user can read the message using the mail command or other mail clients configured to read the local mailbox. To read messages with mail, the user simply needs to type:
```
mail
```

You can also read them directly from the files
```
cat /var/mail/frodo
cat /var/spool/mail/frodo
```

If the email is not delivered, check Postfix's mail log for errors. The mail log is usually located at /var/log/mail.log or /var/log/maillog.
Ensure that Postfix is running: sudo systemctl status postfix.

### Postfix spamming to root user about message delivery error
Postfix sending frequent error messages to the root user about message delivery failures can be both annoying and indicative of an underlying issue that needs to be addressed.

1. Identify the Cause
First, you need to understand why Postfix is generating these error messages. Check the content of the messages to identify common patterns or specific errors that are being reported. Common issues include:
- Configuration errors
- DNS resolution problems
- Network connectivity issues
- Problems with recipient addresses

2. Check the Mail Queue
Inspect the mail queue to see if there are messages that are stuck and causing these notifications
```
postqueue -p
or
mailq
```
This command will list all emails currently in the queue. Look for any messages that are repeatedly failing to be delivered.

3. 1 Redirect Root Emails
If specific errors can't be immediately resolved or you're receiving too many notifications, consider redirecting emails addressed to the root user to another email address where they can be reviewed more conveniently. Edit the /etc/aliases file:
```
sudo vim /etc/aliases
```
And add or modify a line to redirect root's mail:
```
root: your-email@example.com
```
Replace your-email@example.com with the address where you want the messages to go. Then, run the newaliases command to rebuild the alias database:
```
sudo newaliases
```
3. 2  Suppress Non-Delivery Reports (NDRs) for Certain Emails
If certain emails are known to cause issues and don't need to be delivered, you can configure Postfix to discard them or redirect them to a specific address. This involves setting up header checks or transport maps. For example, to discard messages, edit the /etc/postfix/header_checks file:
```
sudo vim /etc/postfix/header_checks
```
And add a rule like:
```
/^Subject:.*error message subject here/ DISCARD
```
Replace error message subject here with a pattern that matches the error emails you're receiving. Then, tell Postfix to use this file by adding the following line to /etc/postfix/main.cf:
```
header_checks = regexp:/etc/postfix/header_checks
```
Reload Postfix to apply the changes:
```
sudo postfix reload
```
