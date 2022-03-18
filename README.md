# Born2beRoot
This project aims to introduce you to the wonderful world of virtualization. You will create your first machine in VirtualBox (or UTM if you can’t use VirtualBox) under specific instructions. Then, at the end of this project, you will be able to set up your own operating system while implementing strict rules.

># Mandatory part
#### 1 : How to set up partitions correctly, watch the video on youtube, link below.
https://youtu.be/EunJ4QJaAEw

#### 2 : Setting Up a Strong Password Policy.
Switch to **root**.
```
su -
```
Install **vim**.
```
apt-get install vim
```
Open file **login.def**.
```
vim /etc/login.defs
```
For password expiration every 30 days, go to line 160.
```
160 PASS_MAX_DAYS   30
```
To set minimum number of days between password changes to 2 days, go to line 161.
```
PASS_MIN_DAYS   2
```
For the user to receive a warning message 7 days before their password expires, go to line 161.
```
PASS_WARN_AGE   7
```
