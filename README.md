# Installing SMTP Server 

This is a basic installation only, 

You can watch the tutorial on YouTube video below for better understanding:

[![Watch on YouTube](https://github.com/SaltyScriptScrap/smtp-rocky9/blob/main/SS_Setup_SMTP_Server.jpg?raw=true)](https://youtu.be/0yA_avdGVwA "Watch on YouTube")


Tested on RockyLinux 9.3, should be compatible to other RHEL / CentOS distributions

What you need:
- a _Gmail account_ with _2-Step Verification_ enabled, 
- an _app password_ created in the Gmail account.



LET'S BEGIN: 

run all commands as sudoers
> `sudo bash`


install the packages needed
> `sudo dnf -y install cyrus-sasl cyrus-sasl-lib cyrus-sasl-plain postfix s-nail`


save the gmail account app password to a file
> `nano /etc/postfix/sasl_passwd`

write your _Gmail Account name_ and _Gmail app password_ you create in this file with this format
```
[smtp.gmail.com]:587 gmail.account.name@gmail.com:gmail.app.password
```
save the file and exit `nano`


set the password file to postfix database
> `postmap hash:/etc/postfix/sasl_passwd`


remove the password file
> `rm -f /etc/postfix/sasl_passwd`


edit the main configuration of postfix
> `nano /etc/postfix/main.cf`

append this to the file
```
relayhost = [smtp.gmail.com]:587
smtp_use_tls = yes
smtp_sasl_auth_enable = yes
smtp_sasl_security_options = noanonymous
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_tls_security_level = secure
smtp_tls_mandatory_protocols = TLSv1.3
smtp_tls_mandatory_ciphers = high
smtp_tls_secure_cert_match = nexthop
smtp_address_preference = ipv4
```
save the file and exit `nano`


enable postfix to start at system startup
> `systemctl enable postfix`


start (or restart) postfix right now
> `systemctl restart postfix`


send your first email 
> `echo "This is the email body " | mail -s "Email subject" -r "from.replyto.address@gmail.com" recipient.address@gmail.com`



# 
# Relaying the SMTP Server
The previous steps can only send email limited from within it self machiche,

Do the next step so other machines can also use this SMTP server

What you need:
- _one or more IP Addresses_ of the other machines to be allowed, for example refer to https://www.postfix.org/BASIC_CONFIGURATION_README.html#relay_from
- one or more _username_ and _password_ on this SMTP Server for sending email from the other machine


let's begin, start by editing the postfix main configuration again,

edit and append the following 
> `nano -w /etc/postfix/main.cf`
```
inet_interfaces = all
# !!!! PLEASE ENTER _one or more IP Addresses_ of the other machines to be allowed below
mynetworks = 192.168.1.0/24
smtpd_relay_restrictions=permit_mynetworks, reject
smtpd_sender_restrictions = permit_mynetworks, reject
smtpd_tls_auth_only = no
smtpd_sasl_auth_enable =   yes
smtpd_sasl_type = cyrus
local_recipient_maps =
unknown_local_recipient_reject_code = 550
smtpd_error_sleep_time = 1s
smtpd_soft_error_limit = 10
smtpd_hard_error_limit = 20
```
save the file and exit `nano`



then edit `master.cf` file, 
replace the first `smtp   inet n   -   n   - - smtpd` line, 
edit with the following
> `nano -w /etc/postfix/master.cf`
```
smtp   inet n   -   n   - - smtpd
      -o smtpd_sasl_auth_enable=yes
      -o smtpd_reject_unlisted_sender=yes
      -o smtpd_recipient_restrictions=permit_sasl_authenticated,reject
      -o broken_sasl_auth_clients=yes
```
save the file and exit `nano`


enable the authentication daemon at startup
> `systemctl enable saslauthd`


start the authentication daemon right now
> `systemctl start saslauthd`


restart postfix right now
> `systemctl restart postfix`


open the machine firewall at port 25 to listen for SMTP request from other machine
> `firewall-cmd --add-port=25/tcp --permanent && firewall-cmd --reload`


DONE, now you can try SMTP from other machines with an assigned username and password from this SMTP Server !
