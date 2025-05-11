
# Setup a mail server with PostFix & Dovecot
This tutorial will describe how to setup a mail server using PostFix, DoveCot and configure a domain on namecheap



## Step 1: Set Hostname & DNS Records

```bash
sudo hostnamectl set-hostname mail.YOURDOMAIN
```
Replace `YOURDOMAIN` with your domain (Eg: mywebsite.com)

Edit `/etc/hosts`:
```bash
127.0.0.1   localhost
X.X.X.X     mail.YOURDOMAIN mail
```
Replace `X.X.X.X` with your server IP

Configure your domain in DNS Provider (in this guide I'm using namecheap.com)
Go to `Account -> Dashboard -> Domain List -> Advanced DNS` set:

In `HOST RECORDS`

- Type: A Record, Host: @, Value: X.X.X.X, TTL: Automatic

- Type: A Record, Host: mail, Value: X.X.X.X, TTL: Automatic

- Type: TXT Record, Host: @, Value: v=spf1 mx ~all, TTL: Automatic

In `MAIL SETTINGS`, select Custom MX

- Type: MX Record, Host: @, Value: mail.`YOURDOMAIN`, 10, Automatic 

## Step 2: Install PostFix + Dovecot
```bash
sudo apt update
sudo apt install postfix dovecot-core dovecot-imapd dovecot-pop3d -y
```
During Postfix install:
- Internet Site
- System mail name: `YOURDOMAIN`
## Step 3: Create Email Users (as Linux users)
For each mailbox, just do
```bash
sudo adduser user1
```
That creates mailboxes like: `user1@YOURDOMAIN` (Eg: user1@mywebsite.com)
Dovecot will deliver to /var/mail/user1

## Step 4: Configure PostFix
Edit `/etc/postfix/main.cf`, add or ensure:
```bash
myhostname = mail.YOURDOMAIN
mydomain = YOURDOMAIN
myorigin = /etc/mailname
inet_interfaces = all
inet_protocols = ipv4
mydestination = $myhostname, localhost.$mydomain, localhost
smtpd_tls_cert_file=/etc/ssl/postfix/cert.pem
smtpd_tls_key_file=/etc/ssl/postfix/key.pem
smtpd_use_tls=yes
smtpd_tls_security_level=may
smtpd_tls_auth_only=yes
smtpd_sasl_type=dovecot
smtpd_sasl_path=private/auth
smtpd_sasl_auth_enable=yes
smtpd_recipient_restrictions=permit_sasl_authenticated,permit_mynetworks,reject_unauth_destination
```
Then restart PostFix:
```bash
sudo systemctl restart postfix
```

## Step 5: Configure Dovecot
Edit `/etc/dovecot/conf.d/10-auth.conf`
```bash
auth_mechanisms = plain login
!include auth-system.conf.ext
```
Edit `/etc/dovecot/conf.d/10-master.conf`, enable
```bash
service auth {
  unix_listener /var/spool/postfix/private/auth {
    mode = 0660
    user = postfix
    group = postfix
  }
}
```
Restart Dovecot
```bash
sudo systemctl restart dovecot
```
## Step 6: Get SSL (Optional)
```bash
sudo apt install certbot -y
sudo certbot certonly --standalone -d mail.YOURDOMAIN
```
Renewal auto-set. Use in Postfix + Dovecot configs as shown earlier.

## Testing
Test sending mail, from the server
```bash
echo "Test message" | mail -s "Test subject" youremail@gmail.com
```
Install `mailutils` for easier mail use
```bash
sudo apt install mailutils
```
To read mail with user1
```bash
su user1
mail
```
