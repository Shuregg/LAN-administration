#

## 0. Prerequirements

* Mail server *Postfix* (Mail Tranfer Agent, MTA)
* IMAP server *Dovecot* 
* fix for maildir (Only for Astra Linux)

    ```bash
    sudo apt-get install postfix dovecot-imapd astrase-fix-maildir
    ```

    ```bash
    dovecot verison: 2.3.4.1 (f79e8e7e4)
    ```

## 1. Pre-settings

### 1.1 New users

    ```bash
    sudo adduser iavmailserver
    sudo adduser iavmailclient
    ```

### 1.2 Mandatory level (skipped)

### 1.3 User's mail files

* Create users mail directories and change access

    ```bash
    sudo -i -u iavmailserver
    maildirmake.dovecot ~/maildir
    exit
    sudo -i -u iavmailclient
    maildirmake.dovecot ~/maildir
    exit

    sudo chown -R iavmailserver:iavmailserver /home/iavmailserver/maildir
    sudo chown -R iavmailclient:iavmailclient /home/iavmailclient/maildir
    ```

## 2. Postfix configuration

### 2.1 Set up Postfix server

* Edit Postfix main config file

    ```bash
    sudo nano /etc/postfix/main.cf
    ```

* Set parameters

    ```bash
    mynetworks = 192.168.122.1/24
    mydomain = iav.miet.stu
    mydestination = $myhostname, localhost, $mydomain
    home_mailbox = maildir/
    smtpd_relay_restrictions = permit_mynetworks, permit_sasl_authenticated, reject_unauth_destinations
    ```

* Restart service

    ```bash
    sudo systemctl restart postfix
    sudo systemctl status postfix
    ```

### 2.2 Dovecot configuration

Set up Dovecot

* Specify type of connetction in *10-auth.conf*

    Open config file

    ```bash
    sudo nano /etc/dovecot/conf.d/10-auth.conf
    ```

    Set parameters

    ```bash
    disable_plaintext_auth = no
    auth_mechanisms = plain login
    ```

* Specify mail directory in *10-mail.conf*

    Open config file

    ```bash
    sudo nano /etc/dovecot/conf.d/10-mail.conf
    ```

    Set parameters

    ```bash
    mail_location = maildir:~/maildir
    ```

    Restart service

    ```bash
    sudo systemctl restart dovecot
    sudo systemctl status dovecot
    ```

### 2.3

    ```bash

    ```