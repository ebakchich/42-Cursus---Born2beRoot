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
- In addition to the root user, a user with your login as username has to be present.
```
visudo
```
add the line below.
```
user_name ALL=(ALL) ALL
```
- This user has to belong to the user42 and sudo groups.
```
groupadd user42
```
add user to the user42 and sudo groups.
```
usermod -aG sudo user_name
```
```
usermod -aG user42 user_name
```
Finally, you have to create a simple script called monitoring.sh. It must be developed in bash.
At server startup, the script will display some information (listed below) on all terminals every 10 minutes (take a look at wall). The banner is optional. No error must
be visible.
Your script must always be able to display the following information:

• The architecture of your operating system and its kernel version.

• The number of physical processors.

• The number of virtual processors.

• The current available RAM on your server and its utilization rate as a percentage.

• The current available memory on your server and its utilization rate as a percentage.

• The current utilization rate of your processors as a percentage.

• The date and time of the last reboot.

• Whether LVM is active or not.

• The number of active connections.

• The number of users using the server.

• The IPv4 address of your server and its MAC (Media Access Control) address.

• The number of commands executed with the sudo program.

FIRST
```
vim /usr/local/bin/monitoring.sh
```
script
```
#!/bin/bash
arc=$(uname -a)
pcpu=$(grep "physical id" /proc/cpuinfo | sort | uniq | wc -l)
vcpu=$(grep "^processor" /proc/cpuinfo | wc -l)
fram=$(free -m | awk '$1 == "Mem:" {print $2}')
uram=$(free -m | awk '$1 == "Mem:" {print $3}')
pram=$(free | awk '$1 == "Mem:" {printf("%.2f"), $3/$2*100}')
fdisk=$(df -Bg | grep '^/dev/' | grep -v '/boot$' | awk '{ft += $2} END {print ft}')
udisk=$(df -Bm | grep '^/dev/' | grep -v '/boot$' | awk '{ut += $3} END {print ut}')
pdisk=$(df -Bm | grep '^/dev/' | grep -v '/boot$' | awk '{ut += $3} {ft+= $2} END {printf("%d"), ut/ft*100}')
cpul=$(top -bn1 | grep '^%Cpu' | cut -c 9- | xargs | awk '{printf("%.1f%%"), $1 + $3}')
lb=$(who -b | awk '$1 == "system" {print $3 " " $4}')
lvmt=$(lsblk | grep "lvm" | wc -l)
lvmu=$(if [ $lvmt -eq 0 ]; then echo no; else echo yes; fi)
#You need to install net tools for the next step [$ sudo apt install net-tools]
ctcp=$(cat /proc/net/sockstat{,6} | awk '$1 == "TCP:" {print $3}')
ulog=$(users | wc -w)
ip=$(hostname -I)
mac=$(ip link show | awk '$1 == "link/ether" {print $2}')
cmds=$(journalctl _COMM=sudo | grep COMMAND | wc -l) # journalctl should be running as sudo but our script is running as root so we don't need in sudo here
wall "	#Architecture: $arc
	#CPU physical: $pcpu
	#vCPU: $vcpu
	#Memory Usage: $uram/${fram}MB ($pram%)
	#Disk Usage: $udisk/${fdisk}Gb ($pdisk%)
	#CPU load: $cpul
	#Last boot: $lb
	#LVM use: $lvmu
	#Connexions TCP: $ctcp ESTABLISHED
	#User log: $ulog
	#Network: IP $ip ($mac)
	#Sudo: $cmds cmd" # broadcast our system information on all terminals
```
next step to run the script, you should be give permission to the script for executable.
 ```
chmod +x /usr/local/bin/monitoring.sh
```
To run script every 10 minutes
```
crontab -u root -e
```
Add at end as follows: (*/10 means every 10 mins the script will show)
```
*/10 * * * * /usr/local/bin/monitoring.sh
```
- SSH & UFW
A SSH service will be running on port 4242 only. For security reasons, it must not be
possible to connect using SSH as root
```
apt install openssh-server
```
Changing default port (22) to 4242
```
vim /etc/ssh/sshd_config
```
Edit the file, change the line #Port22 to Port 4242.
Go to line 15 and edit it.
```
Port 4242
```
It must not be possible to connect using SSH as root.
change line 34.
```
PermitRootLogin no
```
Restart the SSH service.
```
service ssh restart
```
Install UFW
```
apt-get install ufw
```
```
ufw enable
```
Configure the rules
```
ufw allow ssh
```
```
ufw allow 4242
```
Deleting UFW Rules By rule number.
```
ufw status numbered
```
![This is an image](https://blogger.googleusercontent.com/img/a/AVvXsEh-wXgFfnGxCw5CfzsvusVR0vmUAyhzvYcV4xYGW1w2m667uYhDCRzg-ntYwtoovGdYB7KQyYKXfk-WA8Cj-qXoVXHlvb5rO1K9XjFCCQqpX5ncMpP2fyIr2IbmbmLtybLU48kcujcICCyifWD8b-h58MNUvwQR49hNwkOeaof9SRjYrqHc3gLkbYEIeg)
delete the rules number 1 and 2.
```
ufw delete 1
```
```
ufw delete 2
```
- Connecting SSH server.
![This is an image](https://blogger.googleusercontent.com/img/a/AVvXsEgRRTFZwRRR5jwgqLUpVHf1XPBmB0GPXzMZYdddmsCgAkgE5yIGQEuZDDSZCok8RBy3nF4Pb_0Og-rc2mdaT5dt7x1T-HsG6bavyxdqgC3PZ6lk2VIWWw2bo9nuiuJHehp73OgtjFNhFH66I1UBY3yH35kC4lzKHBppI6JwhkFjVR3mAh7SpKq5l3Og0A)
># Bonus part
- Install PHP
- Step 1: Add Surý APT repository to Debian
The DEB.SURY.ORG repository is a home for packaging various software into Debian and Ubuntu based Linux distributions. It contains the latest binary builds of PHP 8.1. The repository has to be added manually on the system.

Install required temporary packages:
```
apt install -y lsb-release ca-certificates apt-transport-https software-properties-common gnupg2
```
Add Surý Debian PPA repository to your Debian system.
```
echo "deb https://packages.sury.org/php/ $(lsb_release -sc) main" | sudo tee /etc/apt/sources.list.d/sury-php.list
```
Import packages signing GPG key;
```
wget -qO - https://packages.sury.org/php/apt.gpg | sudo apt-key add -
```
- Step 2: Install PHP 8.1
Once you’ve added the repository and is confirmed to be functional, we can proceed with the installation of PHP 8.1 o Debian 11/10/9 Linux system.
```
apt update 
```
```
apt install php8.1
```
Install PHP 8.1 Extensions
```
apt install php8.1-{bcmath,fpm,xml,mysql,zip,intl,ldap,gd,cli,bz2,curl,mbstring,pgsql,opcache,soap,cgi}
```
Once the installation is completed,  the Apache web server also installed automatically along with PHP. So, you will need to remove the Apache package from your system.
First, stop the Apache service with the following command:
```
systemctl stop apache2
```
```
systemctl disable apache2
```
Next, remove the Apache package with the following command:
```
apt-get remove apache2  -y
```
- How to Set Up Lighttpd Web Server on Ubuntu.
```
apt-get update -y
```
Once the installation is completed, check the status of Lighttpd with the following command:
```
systemctl status lighttpd
```
