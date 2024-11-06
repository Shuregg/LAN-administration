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

* Add new users

    ```bash
    sudo useradd -m iavmailserver
    sudo passwd iavmailserver

    sudo useradd -m iavmailclient
    sudo passwd iavmailclient
    ```

### 1.2 Mandatory level (skipped)

### 1.3 User's mail files

* Login with new users and create users mail directories

    Or try thee following lines (not verified):

    ```bash
    su - iavmailserver
    mkdir Maildir
    chmod 700 Maildir
    exit

    su - iavmailclient
    mkdir Maildir
    chmod 700 Maildir
    exit
    ```

## 2. Postfix configuration

### 2.1 Set up Postfix server

* Edit Postfix main config file

    ```bash
    sudo nano /etc/postfix/main.cf
    ```

* Set parameters in [main.cf](configs/main.cf) file:

    ```bash
    # See /usr/share/postfix/main.cf.dist for a commented, more complete version
    
    
    # Debian specific:  Specifying a file name will cause the first
    # line of that file to be used as the name.  The Debian default
    # is /etc/mailname.
    #myorigin = /etc/mailname
    
    #smtpd_banner = $myhostname ESMTP $mail_name (AstraLinux)
    #biff = no
    
    # appending .domain is the MUA's job.
    #append_dot_mydomain = no
    
    # Uncomment the next line to generate "delayed mail" warnings
    #delay_warning_time = 4h
    
    #readme_directory = no
    
    # See http://www.postfix.org/COMPATIBILITY_README.html -- default to 2 on
    # fresh installs.
    #compatibility_level = 2
    
    
    
    # TLS parameters
    #smtpd_tls_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem
    #smtpd_tls_key_file=/etc/ssl/private/ssl-cert-snakeoil.key
    #smtpd_use_tls=yes
    #smtpd_tls_session_cache_database = btree:${data_directory}/smtpd_scache
    #smtp_tls_session_cache_database = btree:${data_directory}/smtp_scache
    
    smtp_use_tls = yes
    smtp_sasl_auth_enable = yes
    smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
    smtp_generic_maps = hash:/etc/postfix/generic
    smtp_tls_CAfile = /etc/ssl/certs/Entrust_Root_Certification_Authority.pem
    smtp_tls_session_cache_database = btree:/var/lib/postfix/smtp_tls_session_cache
    smtp_tls_session_cache_timeout = 600s
    smtp_tls_wrappermode = yes
    smtp_sasl_security_options = noanonymous
    smtp_tls_security_level = encrypt
    smtp_tls_loglevel = 1
    
    # See /usr/share/doc/postfix/TLS_README.gz in the postfix-doc package for
    # information on enabling SSL in the smtp client.
    
    #smtpd_relay_restrictions = permit_mynetworks permit_sasl_authenticated defer_unauth_destination
    smtpd_relay_restrictions = permit_mynetworks permit_sasl_authenticated reject_unauth_destination
    #myhostname = server.iav.miet.stu
    myhostname = srv.iav.miet.stu
    #alias_maps = hash:/etc/aliases
    #alias_database = hash:/etc/aliases
    #myorigin = /etc/mailname
    #mydestination = $myhostname, iav.miet.stu, server, localhost.localdomain, localhost
    mydestination = $myhostname, iav.miet.stu, localhost.localdomain, localhost
    relayhost = smtp.yandex.ru:465
    home_mailbox = Maildir/
    #mynetworks = 127.0.0.0/8 [::ffff:127.0.0.0]/104 [::1]/128 192.168.122.0/24
    mynetworks = 192.168.122.0/24
    #mailbox_size_limit = 0
    #recipient_delimiter = +
    #inet_interfaces = all
    #inet_protocols = all
    #inet_protocols = ipv4
    ```

* Restart service

    ```bash
    sudo postfix reload 
    # or
    sudo systemctl restart postfix
    sudo systemctl status postfix
    ```

* Use for debug

    ```bash
    sudo journalctl -u postfix@-.service -r
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
    mail_location = maildir:~/Maildir
    ```

    Restart service

    ```bash
    sudo systemctl restart dovecot
    sudo systemctl status dovecot
    ```

### 2.3

* generic

    ```bash
    mailserver@iav.miet.stu youryandexmail@yandex.ru
    ```

* sasl_passwd

    ```bash
    smtp.yandex.ru youryandexmail@yandex.ru:**********
    ```

* Indexing configs

    ```bash
    sudo postmap /etc/postfix/generic
    sudo postmap /etc/postfix/sasl_passwd
    ```
