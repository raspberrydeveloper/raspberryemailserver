# Raspberry pi email server

## Warning

**This email server will be local only and it will not be able to send email to other providers. (e.g. gmail, hotmail, yahoo etc.)**

## **Prerequisites**

### Hardware

* A raspberry pi.
* An sd card.

### Software

* I suggest to use raspberry pi os because you may have problems with other ones.
* It's also recommended to use ssh because it's easier to copy/paste commands.

## Postfix installation

#### First update and upgrade your os:

``` sh
sudo apt update
```
``` sh
sudo apt upgrade
```
#### Then you can install postifx.

```sh
sudo apt install postfix
```

During the installation, you’ll have to choose these two configuration options:

* The general type of mail configuration: Internet site

* System mail name: domain.com

Just choose internet site and enter your raspberry pi hostname.

Now you will need to make two changes in the configuration that has been generated:

Open the configuration file:

``` sh
sudo nano /etc/postfix/main.cf
```

Enter your raspberry pi hostname name as myhostname:

```
myhostname = domain.com
```

Save and exit (CTRL+O, Enter, CTRL+X).

Restart Postfix:

```sh
sudo service postfix restart
```

At this point, the server should start properly without startup errors.

If this is not the case, look to solve these problems before continuing.

#### Testing

We’ll now make our first test by sending an email from the Raspberry Pi.

For this test, we’ll use telnet to connect to postfix.

Install telnet:

``` sh
sudo apt-get install telnet
```

Connect to the SMTP server:

```sh
telnet localhost 25
```

Enter this series of commands:

```
ehlo 
```

**Warning:** Enter your raspberry pi hostname after the ehlo command.

```
mail from: you@domain.com
```

```
rcpt to: user@mail.com
```

```
data
```
```
Subject: test

Test

.
```
```
quit
```

This commands sequence will create an email and send it to user@mail.com (external email address).

### Mailutils

If you are looking for a most friendly way to do this, you can install mailutils to use the mail command.

* Install mailutils:

```sh
sudo apt install mailutils
```
* Send a test email with the mail command:

```sh
echo 'Test' | mail -s "Test mail command" you@gmail.com
```
In both cases, you can follow the email sending in this log file: /var/log/mail.log

### Receive emails with Postfix

Now it’s time to edit our Postfix configuration to receive emails.

Configuration

We’ll do this by using the Maildir mailboxes format.

Maildir is a safe and easy way to store emails: each mailbox is a directory, and each email is a file.

Edit the configuration file:

```sh
sudo nano /etc/postfix/main.cf
```

Add these lines at the end of the file:

``` sh
home_mailbox = Maildir/
mailbox_command =
```

This configuration will tell Postfix to create a Maildir folder for each system user.

This folder will now host your new incoming emails.

Now we need to create the Maildir folder template by following these steps:

Install this packages:

```sh
sudo apt install dovecot-common dovecot-imapd
```

Create folders in the template directory:

```sh
sudo maildirmake.dovecot /etc/skel/Maildir

sudo maildirmake.dovecot /etc/skel/Maildir/.Drafts

sudo maildirmake.dovecot /etc/skel/Maildir/.Sent

sudo maildirmake.dovecot /etc/skel/Maildir/.Spam

sudo maildirmake.dovecot /etc/skel/Maildir/.Trash

sudo maildirmake.dovecot /etc/skel/Maildir/.Templates
```

These templates will be used when you add new users on your Raspberry Pi.

But for those already existing, you have to do it manually.

For example, you have to run these commands for pi:

```sh
sudo cp -r /etc/skel/Maildir /home/pi/
```
```sh
sudo chown -R pi:pi /home/pi/Maildir
```
```sh
sudo chmod -R 700 /home/pi/Maildir
```
### Testing

You can now repeat the same kind of test as before, but put the user pi in the receiver:

```sh
echo "Test" | mail -s "Test" pi@raspberrypihostname.local
```

And then check that the mail has arrived in the Maildir folder:

```
pi@raspberrypi:~ $ cat /home/pi/Maildir/new/1625109614.Vb302I205f1M127492.raspberrypi
```
```
Return-Path:

X-Original-To: pi@rpi.tips

Delivered-To: pi@rpi.tips

Received: by rpi.tips (Postfix, from userid 1000)

id 1CC5A205F2; Thu,  1 Jul 2021 04:20:14 +0100 (BST)

Subject: Test mail command

To: pi@rpi.tips

X-Mailer: mail (GNU Mailutils 3.5)

Message-Id: 20210701032014.1CC5A205F2@rpi.tips

Date: Thu,  1 Jul 2021 04:20:14 +0100 (BST)

From: pi@raspberrypi

Test
```

You should have only one mail in the new folder, use tab auto-completion to find it.

But we have reached our goal for this step.

We received emails sent to our domain.

## Install Dovecot to allow POP and IMAP connections

We now have a functional mail server,

So, we will move on to the next part, which is to make this mail server accessible to IMAP clients

As you may have noticed, we already installed Dovecot in the previous step to create Maildir folders.

The only thing left to do is to finalize the configuration.

### Configuration

Open the Dovecot mail configuration file:

```sh
sudo nano /etc/dovecot/conf.d/10-mail.conf
```

Edit the Maildir folder:

Replace:

```
mail_location = mbox:~/mail:INBOX=/var/mail/%u
```
With:

```
mail_location = maildir:~/Maildir
```

### Restart Dovecot and Postfix:

``` sh
sudo service postfix restart
```
```sh
sudo service dovecot restart
```

Keep an eye on the log files, and you can use “sudo service X status” to check that the service is running correctly. As I told you previously, a misconfiguration can happen fast. But we’ll do another test now to make sure everything is good.

### Testing

To test the server just open your favorite email client, select advanced configuration and imap4, enter *yourusername@raspberrypihostname.local* as the email and your username and password for username and password. 

Last but not least, select advanced options and enter your *raspberry pi hostname or ip address* for *incoming* and *outgoing* server and select **do not require ssl** for windows mail or **none secuirty type** for gmail on android.
