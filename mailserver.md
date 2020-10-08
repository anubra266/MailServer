# Preparing the Server

## Hostname
We'll use an FQDN:

`mail.anubra.tech`

* mail is the nodename
* anubra.tech is domain name

View the hostname with
```
hostname -f
```

Set the Hostname with
```
sudo hostnamectl set-hostname mail.anubra.tech 
```

Open   `/etc/hosts ` and ensure this line:
```
#   IPv4            FQDN
    199.192.33.73   mail.anubra.tech mail
```

## MX Record
That's pretty Straightforward.🚃

```
@       IN      MX      10      mail
```

## A Record
Also Straightforward. 😉 It must correspond to the **MX Record.**
```
mail    IN      A               199.192.22.73
```
make sure it's your coressponding IPv4 address.

## AAAA Record
If your server uses IPv6 address, it’s also a good idea to add an AAAA record.
Also Straightforward. 😉 It must correspond to the **MX Record.**
```
mail    IN      AAAA            2001:0db8:85a3:0000:0000:8a2e:0370:7334 
```
make sure it's your coressponding IPv6 address.

## PTR Record (Reverse DNS)
If you manage your Nameservers, follow
```
100.0.192.199.in-addr.arpa  IN      PTR     mail
```

# Setting up Postfix

## Installing Postfix

On your server:
```
sudo apt-get update

sudo apt-get install postfix -y
```

* Type of mail configuration; Select ***Internet Site***
* System mail name; Insert your ***domain name*** `anubra.tech`
* Check PostFix veraion with   
  * ```postconf mail_version```
* Install netstat with 
  * ``` sudo apt install net-tools ```
* Open port 25 in firewall with 
  * ```sudo ufw allow 25/tcp```
* Check if port is open with *nmap*
  * ```sudo apt install nmap```
  * ```sudo nmap 199.192.22.73```

## Let's Send Emails
Install Mail Utilities
```
sudo apt-get install mailutils
```
Then Send with 
```
mail anubra266@gmail.com
```

Finish the prompt and check the mail you used above.

## Set postfix hostname
Open config
```
sudo nano /etc/postfix/main.cf
```
modify
```
myhostname = mail.yourdomain.com
```

Restart Postfix
```
sudo systemctl restart postfix
```

## Creating Email Alias

```
sudo nano /etc/aliases
```
add 
```
root:   your-username
```
Rebuild aliases database with 
```
sudo newaliases
```

## Disable IPv6
If you only use IPv4, avoid **wahala** by disabling IPv6
```
sudo postconf -e "inet_protocols = ipv4"
```
Restart PostFix
```
sudo systemctl restart postfix
```

You now have a weel functioning Postfix Server. 😂😇🤣😋


# Dovecot and TLS

## Open all neccessary ports
```
sudo ufw allow 80,443,587,465,143,993/tcp
```
If you use ***POP3*** to fetch emails
```
sudo ufw allow 110,995/tcp
```

## Securing Email Server with TLS Certificate
Install Certbot
```
sudo apt install certbot  python3-certbot-apache
```

Create a virtual host for your ```mail.yourdmain.com```
```
sudo nano /etc/apache2/sites-available/mail.your-domain.com.conf
```
Insert:
```
<VirtualHost *:80>        
        ServerName mail.your-domain.com

        DocumentRoot /var/www/mail.your-domain.com
</VirtualHost>
```

Create the DocumentRoot Directory
```
sudo mkdir /var/www/mail.your-domain.com
```

Set Ownership (Optional)
```
sudo chown www-data:www-data /var/www/mail.your-domain.com -R
```
 ENable it
 ```
 sudo a2ensite mail.your-domain.com.conf
 ```
 Reload Apache
 ```
 sudo systemctl reload apache2
```
Obtain and Install the TLS certificate
```
sudo certbot --apache --agree-tos --redirect --hsts --staple-ocsp --email you@example.com -d mail.your-domain.com
```

## Enable Submission Service in Postfix
Edit Postfix master config
```
sudo nano /etc/postfix/master.cf
```
Edit the Submission Paragraph to be exactly this;
```
submission     inet     n    -    y    -    -    smtpd
 -o syslog_name=postfix/submission
 -o smtpd_tls_security_level=encrypt
 -o smtpd_tls_wrappermode=no
 -o smtpd_sasl_auth_enable=yes
 -o smtpd_relay_restrictions=permit_sasl_authenticated,reject
 -o smtpd_recipient_restrictions=permit_mynetworks,permit_sasl_authenticated,reject
 -o smtpd_sasl_type=dovecot
 -o smtpd_sasl_path=private/auth
 ```

 Ant the smtps so;
 ```
 smtps     inet  n       -       y       -       -       smtpd
  -o syslog_name=postfix/smtps
  -o smtpd_tls_wrappermode=yes
  -o smtpd_sasl_auth_enable=yes
  -o smtpd_relay_restrictions=permit_sasl_authenticated,reject
  -o smtpd_recipient_restrictions=permit_mynetworks,permit_sasl_authenticated,reject
  -o smtpd_sasl_type=dovecot
  -o smtpd_sasl_path=private/auth
  ```
## Tell Postfix path to Certificates
Open main config
```
sudo nano /etc/postfix/main.cf
```
Edit as such
```
smtpd_tls_cert_file=/etc/letsencrypt/live/mail.your-domain.com/fullchain.pem
smtpd_tls_key_file=/etc/letsencrypt/live/mail.your-domain.com/privkey.pem
smtpd_tls_security_level=may 
smtpd_tls_loglevel = 1
smtpd_tls_session_cache_database = btree:${data_directory}/smtpd_scache

smtp_tls_security_level = may
smtp_tls_loglevel = 1
smtp_tls_session_cache_database = btree:${data_directory}/smtp_scache

#Enforce TLSv1.3 or TLSv1.2
smtpd_tls_mandatory_protocols = !SSLv2, !SSLv3, !TLSv1, !TLSv1.1
smtpd_tls_protocols = !SSLv2, !SSLv3, !TLSv1, !TLSv1.1
smtp_tls_mandatory_protocols = !SSLv2, !SSLv3, !TLSv1, !TLSv1.1
smtp_tls_protocols = !SSLv2, !SSLv3, !TLSv1, !TLSv1.1
```

Restart Postfix
```
sudo systemctl restart postfix
```

Check that Postix is listening to required ports
```
sudo netstat -lnpt | grep master
```

You should see Something like;
```
tcp        0      0 0.0.0.0:25              0.0.0.0:*               LISTEN      5098/master         
tcp        0      0 0.0.0.0:587             0.0.0.0:*               LISTEN      5098/master         
tcp        0      0 0.0.0.0:465             0.0.0.0:*               LISTEN      5098/master         
tcp6       0      0 :::25                   :::*                    LISTEN      5098/master         
tcp6       0      0 :::587                  :::*                    LISTEN      5098/master         
tcp6       0      0 :::465                  :::*                    LISTEN      5098/master  
```
If not, something is wrong.

Don't mind any <span style="color:red"> *red colorings* </span>, there's no problem.

## Install Dovecot for your IMAP Server

```
sudo apt install dovecot-core dovecot-imapd
```

And if you use POP3

Also add 
```
sudo apt install dovecot-pop3d
```

Check the version
```
dovecot --version
```

## let's enable IMAP/POP3 Protocol
Edit amin config
```
sudo nano /etc/dovecot/dovecot.conf
```
Add;
```
protocols = imap
```
For POP3'ers
```
protocols = imap pop3
```

Check Mailbox location with 
```
postconf mail_spool_directory
```
You probably get ```mail_spool_directory = /var/mail```

You'll want to change that; Open config:
```
sudo nano /etc/dovecot/conf.d/10-mail.conf
```

Change 
```
 mail_location = mbox:~/mail:INBOX=/var/mail/%u
```

to

```
mail_location = maildir:~/Maildir
```

Confirm or add this line;
```
mail_privileged_group = mail
```

Add dovecot user to mail group
```
sudo adduser dovecot mail
```

## Let's configure Authentication

Open Auth Config
```
sudo nano /etc/dovecot/conf.d/10-auth.conf
```
Uncomment this:
```
sudo nano /etc/dovecot/conf.d/10-auth.conf
```
It will disable plaintext authentication when there’s no SSL/TLS encryption. 

To use full email address (username@your-domain.com) to login, add this to the file.
```
auth_username_format = %n
```
Then Change
```
auth_mechanisms = plain
```
to
```
auth_mechanisms = plain login
```

## Configuring SSL/TLS Encryption

Edit SSL/TlS Config
```
sudo nano /etc/dovecot/conf.d/10-ssl.conf
```
Change 
```
ssl = yes
```
to
```
ssl = required
```

Specify path to SSL Certificate
```
ssl_cert = </etc/letsencrypt/live/mail.your-domain.com/fullchain.pem
ssl_key = </etc/letsencrypt/live/mail.your-domain.com/privkey.pem
```

change
```
ssl_prefer_server_ciphers = no
```
to
```
ssl_prefer_server_ciphers = yes
```

## SASL Authentication Between Postfix and Dovecot

Edit
```
sudo nano /etc/dovecot/conf.d/10-master.conf
```
Change the ```service auth``` section to;
```
service auth {
    unix_listener /var/spool/postfix/private/auth {
      mode = 0660
      user = postfix
      group = postfix
    }
}
```
## Auto-create Sent and Trash Folder
Edit Mailbox config
```
sudo nano /etc/dovecot/conf.d/15-mailboxes.conf
```
add
```
auto = create
```
to Drafts, Junk, Trash and Sent. Such as;
```
 mailbox Trash {
    auto = create
    special_use = \Trash
 }
 ```
 Restart Dovecot
 ```
 sudo systemctl restart dovecot
```
Check that Dovecot is listening on port 143 (IMAP) and 993 (IMAPS).
```
sudo netstat -lnpt | grep dovecot
```
It would look like
```
tcp        0      0 0.0.0.0:4190            0.0.0.0:*               LISTEN      802/dovecot         
tcp        0      0 0.0.0.0:993             0.0.0.0:*               LISTEN      802/dovecot         
tcp        0      0 0.0.0.0:995             0.0.0.0:*               LISTEN      802/dovecot         
tcp        0      0 0.0.0.0:110             0.0.0.0:*               LISTEN      802/dovecot         
tcp        0      0 0.0.0.0:143             0.0.0.0:*               LISTEN      802/dovecot         
tcp6       0      0 :::4190                 :::*                    LISTEN      802/dovecot         
tcp6       0      0 :::993                  :::*                    LISTEN      802/dovecot         
tcp6       0      0 :::995                  :::*                    LISTEN      802/dovecot         
tcp6       0      0 :::110                  :::*                    LISTEN      802/dovecot         
tcp6       0      0 :::143                  :::*                    LISTEN      802/dovecot  
```

If not, something is wrong.

Don't mind any <span style="color:red"> *red colorings* </span>, there's no problem.

If there’s a configuration error, dovecot will fail to restart, so it’s a good idea to check if Dovecot is running with the following command.

```
systemctl status dovecot
```
Restart Postfix
```
sudo systemctl restart postfix
```
## Using Dovecot to Deliver Email to Message Store

Install Dovecot LMTP Server
```
sudo apt install dovecot-lmtpd
```
Edit Main Config 
```
sudo nano /etc/dovecot/dovecot.conf
```
Add lmtp to protocols
```
protocols = imap lmtp
```
Edit master config
```
sudo nano /etc/dovecot/conf.d/10-master.conf
```
Change ```lmtp service``` to
```
service lmtp {
 unix_listener /var/spool/postfix/private/dovecot-lmtp {
   mode = 0600
   user = postfix
   group = postfix
  }
}
```

Now Edit Postfix config
```
sudo nano /etc/postfix/main.cf
```

Add these lines
```
mailbox_transport = lmtp:unix:private/dovecot-lmtp
smtputf8_enable = no
```

Save and Close, the restart the services
```
sudo systemctl restart postfix dovecot
```

## Auto-Renew TLS Certificate

```
sudo crontab -e

@daily certbot renew --quiet && systemctl reload postfix dovecot apache2
```

## Configure Email Client

Now open up your desktop email client such as Mozilla Thunderbird and add a mail account.

* In the incoming server section, select IMAP protocol, enter mail.your-domain.com as the server name, choose port 143 and STARTTLS. Choose normal password as the authentication method.
  
* In the outgoing section, select SMTP protocol, enter mail.your-domain.com as the server name, choose port 587 and STARTTLS. Choose normal password as the authentication method.

```
Hint: You can also use port 993 with SSL/TLS encryption for IMAP, and use port 465 with SSL/TLS encryption for SMTP.
```

To add users;
```
sudo adduser user1
```

## Disable Receiving Email

Open Config
```
sudo nano /etc/postfix/main.cf
```
Change;
```
inet_interfaces = all
```
to
```
inet_interfaces = loopback-only
```
and Restart Postfix 
```
sudo systemctl restart postfix
```

# Let's avoid Spam

## Create SPF record
```
@       IN  TXT         "v=spf1 mx ip4:199.192.22.73 ~all"
```

## Setup DKIM

Open Signing Table file
```
sudo nano /etc/opendkim/signing.table
```

list domains there
```
*@domain1.com       default._domainkey.domain1.com
*@domain2.com       default._domainkey.domain2.com
```
where default is the *selector.*

Edit Table key file
```
sudo nano /etc/opendkim/key.table
```
Add domains again
```
default._domainkey.domain1.com     domain1.com:default:/etc/opendkim/keys/domain1.com/default.private
default._domainkey.domain2.com     domain2.com:default:/etc/opendkim/keys/domain2.com/default.private
```
Edit the trusted hosts
```
sudo nano /etc/opendkim/trusted.hosts
```

Should be like
```
127.0.0.1
localhost

*.domain1.com
*.domain2.com
```

Make Key directories for all or your one domain
```
sudo mkdir /etc/opendkim/keys/your-domain.com
```

Generate Key
```
sudo opendkim-genkey -b 2048 -d your-domain.com -D /etc/opendkim/keys/your-domain.com -s default -v
```

Make openkdim the owner of the key
```
sudo chown opendkim:opendkim /etc/opendkim/keys/domain2.com/default.private
```

Display the key
```
sudo cat /etc/opendkim/keys/domain2.com/default.txt
```

You get something like
```
default._domainkey	IN	TXT	( "v=DKIM1; h=sha256; k=rsa; "
	  "p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAriOBzw1quiaErdWGT1GCt+ybGQv4iGvuhhWUciUH1EzMvQZp1eXK+jTGreuRYwgDbF2dddjwijiI8wdj8nj91vp2DR+38slm+Mj6qTPUKM3n/ctP5L3Q3ymhBhpzGrXpdAAg2711SwYqR42C4jBu84LIQMIGAa3HVsiyq8/gtpd+5RlKJhjE74xdHPTGWoBZ4WZhMWlOPXDePpoH1"
	  "zrLrP2TuUUikq4lkNeSnqIRVatjk/pVtBczgkYBsn07IeN0tLA1WxzcKHWUdMg+u0COvGgy6mwaGg9Een/ElpjoDQKRQlNBADvIRfA41aWWRH48/5VM6vaWBphi/3NtWJxVH6t+QIDAQAB" )  ; ----- DKIM key default for your-domain.com
```

The string after the p parameter is the public key.

Now Create a TXT record like so:
```
default._domainkey	IN	TXT	    ( "v=DKIM1; h=sha256; k=rsa; "
	  "p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAriOBzw1quiaErdWGT1GCt+ybGQv4iGvuhhWUciUH1EzMvQZp1eXK+jTGreuRYwgDbF2dddjwijiI8wdj8nj91vp2DR+38slm+Mj6qTPUKM3n/ctP5L3Q3ymhBhpzGrXpdAAg2711SwYqR42C4jBu84LIQMIGAa3HVsiyq8/gtpd+5RlKJhjE74xdHPTGWoBZ4WZhMWlOPXDePpoH1"
	  "zrLrP2TuUUikq4lkNeSnqIRVatjk/pVtBczgkYBsn07IeN0tLA1WxzcKHWUdMg+u0COvGgy6mwaGg9Een/ElpjoDQKRQlNBADvIRfA41aWWRH48/5VM6vaWBphi/3NtWJxVH6t+QIDAQAB" )
```
Restart bind and Check the TXT Record
```
sudo service bind9 restart
dig TXT default._domainkey.domain2.com
```
 Check if DKIM DNS is correct
```
sudo opendkim-testkey -d domain2.com -s default -vvv
```
Restart OpenDKIM
```
sudo systemctl restart opendkim
```
## Create DMARC Record
```
v=DMARC1; p=none; pct=100; rua=mailto:dmarc-reports@your-domain.com
```
