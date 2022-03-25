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
PASS_MAX_DAYS   30
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
go to line 25 and modify it as in the example below.
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
- After setting up your configuration files, you will have to change
all the passwords of the accounts present on the virtual machine,
including the root account.
 #### 3 : Configuring sudo.
 To set up a strong configuration for your sudo group, you have to comply with the
following requirements:
```
apt install sudo
```
Open file **sudoers** using vim.
```
vim /etc/sudoers
```
- Authentication using sudo has to be limited to 3 attempts in the event of an incorrect password.
go to line 12 and add the line below.
```
Defaults     passwd_tries=3
```
- A custom message of your choice has to be displayed if an error due to a wrong
password occurs when using sudo.
go to line 13 and add the line below.
```
Defaults     badpass_message="Password is wrong, please try again!"
```
- Each action using sudo has to be archived, both inputs and outputs. The log file
has to be saved in the /var/log/sudo/ folder.
First, create a file, name it as you like with the extension .log
```
touch /var/log/sudo/arv.log
```
Then
```
vim /etc/sudoers
```
go to line 14 and add the line below.
```
Defaults	logfile="/var/log/sudo/arv.log"
Defaults	log_input,log_output
```
- The TTY mode has to be enabled for security reasons.
go to line 16 and add the line below.
```
Defaults        requiretty
```
- For security reasons too, the paths that can be used by sudo must be restricted.
go to line 11 and modify it as in the example below.
```
Defaults   secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"
```
- Finally, you have to create a simple script called monitoring.sh. It must be developed in bash.
