# Cyberdefenders.org - First CTF
This was the first ctf organized by Cyberdefenders.org - the first blueteam ctf, it-sec.fail has participated.
It was quite of fun and know you can play it for your self! Try it on [Cyberdefenders.org](https://www.cyberdefenders.org)

## Just a few words
I will use linux to work on this tasks, but you can also use windows. The tools listet in the [Scenario](#scenario) should be available under windows.
In Linux I will use the ewf-tools to work with the E01 files. With the ewf-tools I can mount the E01 image and work with my linux tools on the mounted filesystem of the
os-image.

## Scenario

The enterprise EDR alerted for possible exfiltration attempts originating from a developer RedHat Linux machine. A fellow SOC member captured a disk image for the suspected machine and sent it for you to analyze and identify the attacker's footprints.
Tools

 - [FTKImager](https://accessdata.com/product-download/ftk-imager-version-4-5)
 - [R-studio](https://www.rstudio.com/)
 - [JohnTheRipper](https://www.openwall.com/john/)
 - [hashcat](https://hashcat.net/hashcat/)

## Prerequisites
First of all, we have to mount the E01 image on the machine, to get our hands dirty.

### Windows
I don't use windows, and quite often it is difficult (running vba script or something like that). But I can't tell, how to work on this on windows.
But maybe I can pursuade someone from the team for explaining it on windows :-)

### Linux
In this case I can help you :-)

Sometimes it is necessary to be superuser :-)
First of all we have to mount the E01 Image, therefore I'll create to directories.
```shell
mkdir ewf
mkdir mount
```
Now we can mount the ewf file there and the `mount` directory can be used to mount the filesystem. Mount the E01 file with `ewfmount`

```shell
$ ewfmount 02.E01 ewf/
ewfmount 20140608

$ mount ewf/ewf1 mount/
```
Sometimes it is necessary to look at the Imagefile, in cases you have an image from a lvm device or something like that.
In this case you should see something like this, when you look in `mount/`

```shell
$ ls mount/
bin  boot  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
```

Okay everything is prepared, lets go to the challenges, shall we?

## Tasks

### 1 - What is the RHEL version installed on the machine?
This one is quite easy, just look at the release file.

``` shell
$ cat etc/*release*
NAME="Red Hat Enterprise Linux"
VERSION="8.4 (Ootpa)"
ID="rhel"
ID_LIKE="fedora"
VERSION_ID="8.4"
PLATFORM_ID="platform:el8"
PRETTY_NAME="Red Hat Enterprise Linux 8.4 (Ootpa)"
ANSI_COLOR="0;31"
CPE_NAME="cpe:/o:redhat:enterprise_linux:8.4:GA"
HOME_URL="https://www.redhat.com/"
DOCUMENTATION_URL="https://access.redhat.com/documentation/red_hat_enterprise_linux/8/"
BUG_REPORT_URL="https://bugzilla.redhat.com/"

REDHAT_BUGZILLA_PRODUCT="Red Hat Enterprise Linux 8"
REDHAT_BUGZILLA_PRODUCT_VERSION=8.4
REDHAT_SUPPORT_PRODUCT="Red Hat Enterprise Linux"
REDHAT_SUPPORT_PRODUCT_VERSION="8.4"
Red Hat Enterprise Linux release 8.4 (Ootpa)
Red Hat Enterprise Linux release 8.4 (Ootpa)
cpe:/o:redhat:enterprise_linux:8.4:ga
```
Okay it is an RHEL 8.4

### 2 - How many users have a login shell?
For this task we can count the user with a login shell in the passwd file.
Okay then:

```shell
$ cat etc/passwd | grep -v nologin
root:x:0:0:root:/root:/bin/bash
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
cyberdefenders:x:1000:1000:cyberdefenders:/home/cyberdefenders:/bin/bash
rossatron:x:1001:1001::/home/rossatron:/bin/bash
chandler:x:1002:1002::/home/chandler:/bin/bash
tribbiani.j:x:1003:1003::/home/tribbiani.j:/bin/bash
rachel:x:1004:1004:Anon:/home/rachel:/bin/bash
```
Okay there are some unwanted results left. But as we see, we can grep for the bash shell instead of inverting.

This command should get some answers. `$ cat etc/passwd | grep bash | wc -l`

### 3 - How many users are allowed to run the sudo command on the system?
Lets have a look at the sudoers-file `$ cat etc/sudoers`

> Snippet from sudoers-file:

```text
## Allow root to run any commands anywhere 
root    ALL=(ALL)   ALL

## Allows members of the 'sys' group to run networking, software, 
## service management apps and more.
# %sys ALL = NETWORKING, SOFTWARE, SERVICES, STORAGE, DELEGATING, PROCESSES, LOCATE, DRIVERS

## Allows people in group wheel to run all commands
%wheel  ALL=(ALL)   ALL

## Same thing without a password
# %wheel    ALL=(ALL)   NOPASSWD: ALL
```

Okay, we have to take a look at the groups aswell, but beside the root-user, only the `wheel` group can use sudo.
It is not a good idea, to let user use sudo without password ;-) - just a short sidenote.

To get the flag, we need to know, how many users can use sudo, so lets examine the `wheel` group

```shell
$ cat etc/group | grep wheel
wheel:x:10:cyberdefenders,rachel
```

### 4 - What is the 'rossatron' user password?
To get tha password of the user, we could look at the bash_history from the user - sometimes user accedently type the password into the shell.
But we will not have so much luck ;-).

I'll take a different approach and will use johntheripper.

Therfore we need to unshadow the passwd and shadow entry for the rossatron user.

```shell
$ unshadow etc/passwd etc/shadow  | grep rossatron > ../passwd_rossatron_for_john

$ john --wordlist=../rockyou.txt ../passwd_rossatron_for_john 
Warning: detected hash type "sha512crypt", but the string is also recognized as "HMAC-SHA256"
Use the "--format=HMAC-SHA256" option to force loading these as that type instead
Warning: detected hash type "sha512crypt", but the string is also recognized as "sha512crypt-opencl"
Use the "--format=sha512crypt-opencl" option to force loading these as that type instead
Using default input encoding: UTF-8
Loaded 1 password hash (sha512crypt, crypt(3) $6$ [SHA512 128/128 AVX 2x])
Cost 1 (iteration count) is 5000 for all loaded hashes
Will run 16 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
0g 0:00:00:14 0,43% (ETA: 21:23:20) 0g/s 5141p/s 5141c/s 5141C/s poopbutt..compu
0g 0:00:00:17 0,51% (ETA: 21:24:05) 0g/s 5113p/s 5113c/s 5113C/s 100677..pinky88
0g 0:00:00:18 0,54% (ETA: 21:24:04) 0g/s 5105p/s 5105c/s 5105C/s melissam..doidinha
0g 0:00:00:19 0,57% (ETA: 21:24:02) 0g/s 5098p/s 5098c/s 5098C/s erica18..Dominic1
0g 0:00:00:21 0,63% (ETA: 21:24:03) 0g/s 5076p/s 5076c/s 5076C/s barina..192519
0g 0:00:00:22 0,66% (ETA: 21:24:31) 0g/s 5068p/s 5068c/s 5068C/s iordache..chaina
rachelgreen      (rossatron)
1g 0:00:00:59 DONE (2021-11-08 20:29) 0.01682g/s 4980p/s 4980c/s 4980C/s redson..pookies1
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```

Thanks for your help john!

### 5 - What is the Victim's IP address?
To get a IP address of an user that logged into the system, you can look at different locations.
 - secure log
 - audit.log
 - lastlog (strings)
 - wtmp (via `last -f`)

I will have a look into the `/var/log/secure` logfile to get the whole package.
There is one entry of a login from a different location:

```text
Aug 24 08:22:58 localhost systemd[2224]: pam_unix(systemd-user:session): session opened for user cyberdefenders by (uid=0)
Aug 24 08:22:58 localhost gdm-password][2213]: pam_unix(gdm-password:session): session opened for user cyberdefenders by (uid=0)
Aug 24 08:23:01 localhost polkitd[1030]: Registered Authentication Agent for unix-session:2 (system bus name :1.261 [/usr/bin/gnome-shell], object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale en_
US.UTF-8)
Aug 24 08:23:08 localhost polkitd[1030]: Unregistered Authentication Agent for unix-session:c1 (system bus name :1.57, object path /org/freedesktop/PolicyKit1/AuthenticationAgent, locale en_US.UTF-8) (disconnect
ed from bus)
Aug 24 08:23:08 localhost gdm-launch-environment][1446]: pam_unix(gdm-launch-environment:session): session closed for user gdm
Aug 24 08:23:59 localhost sshd[2979]: Connection from 192.168.196.128 port 48762 on 192.168.196.129 port 22
Aug 24 08:24:19 localhost sshd[2979]: Accepted key RSA SHA256:mVT+DmLq2ctDhRYn7DrSN7a7TBGpyLeKnC2ZQgPDsjQ found at /home/chandler/.ssh/authorized_keys:1
Aug 24 08:24:19 localhost sshd[2979]: Postponed publickey for chandler from 192.168.196.128 port 48762 ssh2 [preauth]
Aug 24 08:24:19 localhost sshd[2979]: Accepted key RSA SHA256:mVT+DmLq2ctDhRYn7DrSN7a7TBGpyLeKnC2ZQgPDsjQ found at /home/chandler/.ssh/authorized_keys:1
Aug 24 08:24:19 localhost sshd[2979]: Accepted publickey for chandler from 192.168.196.128 port 48762 ssh2: RSA SHA256:mVT+DmLq2ctDhRYn7DrSN7a7TBGpyLeKnC2ZQgPDsjQ
Aug 24 08:24:20 localhost systemd[2986]: pam_unix(systemd-user:session): session opened for user chandler by (uid=0)
Aug 24 08:24:20 localhost sshd[2979]: pam_unix(sshd:session): session opened for user chandler by (uid=0)
```

In this lines you will get the flag :-)

### 6 - What service did the attacker use to gain access to the system?
This one should be quite easy, because we analyzed the logs a task ago ;-)

### 7 - What is the attacker's IP address?
Look at the same logfile as for task five. This IP has a massive amount of connections to the system.

### 8 - What authentication attack did the attacker use to gain access o the system?
Again, look at the same logs. You'll see a massive amount on login attempts and wrong password failures.
What kind of attack could that be? :D 

### 9 - How many users the attacker was able to bruteforce their password?
Same log again, count the successfull login attempts after the first attack.
The logentrys you are looking for is `Accepted password`.

You could count them via wc -l with something like:
```shell
grep "Accepted password" var/log/secure | awk '{print $9,"    ",$11 }' | uniq | grep 192.168.196.128 
# the awk will look for the attacker ip and the username
# the uniq will... unique the output :D 
```
But this result is not the answer! You have to check the failed logins from this useraccounts as well.
Check if the user were bruteforced or if some of them are logged in over another way ;-)

### 10 - When did the attack start? (DD/MM/YYYY)
Look at the secure log for this as well! Its end of august. The snipped from task 5 will not help you.

### 11 - What is the user used by the attacker to gain initial access to system?
To answer this question, you can also use the secure logfile and look for the first succsessful log

### 12 - What is the MITRE ID of the technique used by the attacker to achieve persistence?

### 13 - What is the CVE number used by the attacker to escalate his Privilege?

### 14 - After gaining more privilege the attacker dropped a backdoor to gain more persistence which receives commands from Gmail account. What is the email used to send commands?

### 15 - The attacker downloaded a keylogger to capture users' keystrokes. What is the secret word the attacker was able to exfiltrate?

