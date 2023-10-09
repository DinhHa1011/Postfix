# Postfix report 
# Table of contents

- [Overview](#overview)
- [Install](#install)
- [Use](#use)
- [Set up automatic send email](#set-up-automatic-send-email)
- [References](#references)
## Overview
- Pflogsumm is designed to provide an over-view of postfix activity, with just enough detail to give the administrator 
## Install 
- Update Ubuntu 
```
sudo apt update 
```
- Install Pflogsumm and mutt (for send mail report)
```
sudo apt install pflogsumm mutt -y
```
## Use
- Generate a report for today.
```
sudo pflogsumm -d today /var/log/mail.log
```
- Generate a report for yesterday.
```
sudo pflogsumm -d yesterday /var/log/mail.log
```
## Set up automatic send email 
- Method: Crontab
- Open crontab config
```
sudo crontab -e
```
- Add the following line, which will send a report every day at 8:00 AM to admin@example.com (change it to your email)
```
0 8 * * * /usr/sbin/pflogsumm -d yesterday /var/log/mail.log --problems-first --rej-add-from --verbose-msg-detail -q | mutt -s "Postfix log summary"  admin@example.com
```

## References 
* [Analyze Postfix Logs](https://www.linuxbabe.com/mail-server/configure-postscreen-in-postfix-to-block-spambots#:~:text=Step%205%3A%20Analyze%20Postfix%20Logs)
