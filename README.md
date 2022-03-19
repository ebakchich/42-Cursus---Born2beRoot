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
Open file **login.def** using vim.
```
vim /etc/login.defs
```
For password expiration every 30 days, go to line 160 and edit it.
```
160 PASS_MAX_DAYS   30
```
To set minimum number of days between password changes to 2 days, go to line 161 and edit it.
```
PASS_MIN_DAYS   2
```
For the user to receive a warning message 7 days before their password expires, go to line 162 and edit it.
```
PASS_WARN_AGE   7
```
To set up policies in relation to password strength, install the libpam-pwquality package.
```
apt install libpam-pwquality
```
Open file **common-password** using vim.
```
vim /etc/pam.d/common-password
```
go to line 25 and edit modify it as in the example below.
```
password        requisite                       pam_pwquality.so retry=3 minlen=10 ucredit=-1 dcredit=-1 maxrepeat=3 reject_username enforce_for_root difok=7
```
**minlen** = minimum password length.

**ucredit** = maximum number of uppercase characters that will generate a credit.

**dcredit** = maximum number of digits that will generate a credit.

**maxrepeat** = the maximum number of times a single character may be repeated.

**reject_username** = The option rejects a password if it consists of the username either in its normal way or in reverse.

**enforce_for_root** = This ensures that the password policies are adhered to even if it’s the root user configuring the passwords.

**difok** = the minimum number of characters that must be different from the old password.
Configuring sudo
- [x] After setting up your configuration files, you will have to change
all the passwords of the accounts present on the virtual machine,
including the root account.
 #### 3 : Configuring sudo.
 To set up a strong configuration for your sudo group, you have to comply with the
following requirements:
- Authentication using sudo has to be limited to 3 attempts in the event of an incorrect password.
