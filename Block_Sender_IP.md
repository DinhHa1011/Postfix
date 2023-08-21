
# Blocking sender IPs in Postfix
# Table of contents

- [Blocking sender IPs in Postfix](#blocking-sender-ips-in-postfix)
  - [Create IP Blacklist](#create-ip-blacklist)
  - [Link to postfix](#link-to-postfix)
  - [Result](#result)
  - [References](#references)
## Create IP Blacklist
- /etc/postfix/client_checks 
```
vim client_checks 
```
```
123.456.789.123           REJECT Your IP is spam
123.456.789.0/24          REJECT Your IP range is spam
```
- Hash the blacklist
```
postmap /etc/postfix/client_checks
```
## Link to postfix 
- /etc/postfix/main.cf 
```
vim /etc/postfix/main.cf 
```
```

smtpd_recipient_restrictions =
   check_client_access cidr:/etc/postfix/client_checks,
```
- Restart Postfix
```
systemctl restart postfix
```
## Result 
![](https://i.imgur.com/w1jGEU7.png)
## References
* [Blocking sender IPs in Postfix](https://ethitter.com/2016/03/blocking-sender-ips-in-postfix/)
