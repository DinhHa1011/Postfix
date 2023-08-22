
# Allow specific IP or Range IPs send mail in Postfix
# Table of contents

- [Allow specific sender IPs in Postfix](#allow-specific-sender-ips-in-postfix)
- [Table of contents](#table-of-contents)
  - [Create IP Allow](#create-ip-allow)
  - [Link to postfix](#link-to-postfix)
  - [Result](#result)
  - [References](#references)

## Create IP Allow List
- /etc/postfix/client_checks
```
vim /etc/postfix/client_checks 
```
```
#Ip or range allow
58.186.68.93    OK
#Block all
0.0.0.0/0       REJECT Dia Chi IP Khong Duoc Phep Gui Mail

```
- Hash the list
```
postmap /etc/postfix/client_checks  
```
## Link to postfix 
- /etc/postfix/main.cf 
```
vim /etc/postfix/main.cf 
```
```
mua_client_restrictions =
        check_client_access cidr:/etc/postfix/client_checks,
        reject
```
- /etc/postfix/master.cf
In `submission inet n       -       y       -       -       smtpd` 
Uncomment 
```
  -o smtpd_client_restrictions=$mua_client_restrictions

```
- Restart Postfix
```
systemctl restart postfix
```
## Result 
- Listed IP :**58.186.68.93** -  OK
![](https://i.imgur.com/86Iu6K4.png)
- Not listed IP : **104.28.222.74** - Block
![](https://i.imgur.com/QbFYg7F.png)

## References
* [How to only accept e-mail from a single IP address](https://askubuntu.com/questions/490761/how-to-only-accept-e-mail-from-a-single-ip-address)
* [Restrictions for postfix submission port](https://serverfault.com/questions/728682/restrictions-for-postfix-submission-port)
