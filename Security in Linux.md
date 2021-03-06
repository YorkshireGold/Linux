### MAC( Mandatory access Control) and DAC ( Discretionary Access Control )

Discretionary access Controls is a means of restricting access to objects based on the identity of the subject and the groups with which they belong. The discretion is from subjects that have certain permissions, being capable to pass permissions on to other subjects. Thats what users,groups and permission are all doing. 

In Linux, two DAC mechanism are used
- Owner-based permission together with file modes
- Capability Systems 

Owner-based permission : UGO ( User , Group and Others). Every file has an owner and every entity that is UGO has user group permissions applied to it. 

permissions include: Read, Write and execute. 

There are also some specials to meet specific needs

- SUID (Set User ID): only has a meaning on a file. It allows a user to run a file as the owner of the file and not as them selves, so SUID == root will alow a user to be the root of the system while that file is running .

- SGID (Set Group ID ): same as above but for the group ownership. 

- StickyBit: only has effect on directories and if it is set on a file or directory only allows the deletion of that object by the owner of the item ( file , directory).

- ACL ( Access controls Lists) : Created for setting default permissions and to specify multiple owners.

- Attributes: These are properties that an admin can set on a file. 

### Capabilities

Capabilities are settings which give a files and software different access controls for the task they need to do.  

### MAC

The big difference MAC is the operating system enforces, and there is nothing the user can do about it; and thats why MAC is stringer than DAC on Linux.

This is implemented in the way the operating system constrains the ability of a subject or initiator to access or generally preform some sort of operation on an object or target. Subjects are typically users, processes or threads. Targets are typically files, directories, network ports , shared memory segments, IO devices. The subjects as well as the targets have a set of security attributes. Based on these authorization rules, the OS kernel can examine the attributes and decide if access is allowed by a subject to a target or not. 

The leading standard of MAC solutions on Linux is SELinux. SELinux originates from Red HAt, currently available of most distributions and co-developed with the NSA (apparently) but still open source. 

Smack is a MAC solutions set up for embedded systems. 

In SELinux...
setenforce permissive # Allows everything but still logs it all
setenforce Enforcing # turns SELinux back on for 


```sh
su           # The switch user command , without any arguments will ask for the root password and then switch you to the root user. 
su -         # Same as above but wil open a new shell with the root environment variables
sudo -i      # opens a rot shell
ssh-copy-id 
w           # Not a typo, type w to see who is logged in on the system
```

To configure the sudoers for the system , use the command ```sudo visudo``` .

## Working with Users and groups

groupadd (create groups)

adduser (Ubuntu) & useradd (CentOS)

```sh
adduser  bob                # add bob as a user ( UBUNTU only)
usermod --help              # Lots of options including locking and unlocking accounts
groupadd <GROUP_NAME>
userdel bob                 # delete bob as a user
useradd - D                 # displays the default setting for new user adds
/etc/skell                  # Contents is copied to use home directory upon user creation
/etc/login.defs             # used as the default configuration so changing this file will change the defaults. Such as password length, GRoup id, home dir
passwd -l <USER>            # Locks the password
passwd -u <USER>            # UNLOCKS the password
passwd -S <USER>            # Returns the status of the password
chage <USER>                # will load up the process to set password lifetime settings - Useful for admins
find / -perm /2000 # find files that have SGID
find / -perm /1000 # find files that have Sticky bit
find / -perm /4000 # find files that have SUID
```

There are four files for configuring centralised group and user information
/etc/shadow
/etc/group
/etc/gshadow
/etc/passwd

`/etc/passwd` - this is historically the file tha Unix has used to store user information. As this file is normally set to read for all users, the passwords are no longer stored in here. Instead they are encrypted and stored in hte `/etc/shadow`. Groups are stored in `/etc/group` . `/etc/gshadow` is not used anymore but it is a legacy file to set passwords for groups. You can modify the users and groups the safest way is to use `vipw` which will open a temporary file of `/etc/passwd` in VI. This will prevent cases where other users aer in `useradd`. OT do the same for `/etc/shadow`, use `vipw -s`. `vigr` will let you edit groups.

/etc/login.defs


# Linux Operating System Security
## Keeping up-to-date
CentOS `yum updateinfo` - this will run an update on the packages
RHEL `yum updateinfo` - the same command on RHEL will just give the info with an advisor code "RHSA-2016:0176"

### UBUNTU 
Ubuntu `apt-get -s dist-upgrade` - shows a list of all the updates that are available for a system

 `apt-get -s dist-upgrade | grep "^inst" | awk -F " " '{print $2}' | xargs apt-get install` this will list all the packages that are installers, get the names of them and then ppe them to the installer.

 `unattended-upgrades` will command will run a hands off use to upgrade good for "domestic" use but this is not good in cooporate environments as the updates need to be tested etc. 

 ### Validateing packages

On RHEL type distros the simplest way to validate all packages is with the rpm package managere. Use the command `rpm -Va` will verify.

On Ubuntu `apt install debsums` will insall the right package to check the hashsums of the packages.
firstly generae a list of packages checksums with the command `debsums -l` and then run a check on those with the comand `debsums -c` to check.

### <ins>AIDE (Advanced Intrusion Detection Environment)</ins>

Aide is for scanning the static parts of your server/file system as it will look for changes. Don't use it on places where things change a lot as it will raise lots of false positives. 

To install on RHEL run `yum install -y aide`. 

Configuration file `vim /etc/aide.conf`. In this file the `database` ( which the contens of scan targets set in the configuration) can be changed on the place on where to look.

In the configuration, after the options you will see profiles such as 
```
# NORMAL = R+sha512
NORMAL = p+i+n+u+g+s+m+c+acl+selinux+xattrs+sha512
```
This line has many options separated by the '+' sign.

further on there a lots of listed locations for different targets to go into the database. 

To run **AIDE**
```
aide --init
```
After a few minutes yor scan will produce this report file `/var/lib/aide/aide.db.new.gz`.
The next time the scan runs it wil create another file with exactly the same name so the best thing to do is to **move this report to external storage and rename it with a unique date time.**

Once we have made our first scan and perhaps made some trivial change to the system like add a new user, you can run a comparison check with `aide --check` which is also the default command if you just type `aide`.

## Manageing file system security properties

### Createing encrypted block devices

`Luks` si the Linux encryption layer.

You need an empty device, for example `/dev/sdb1`. 
Format the device
Luks open will create a new luks device that you are going ot work with. Within this new luks device is where you make the new file system. Once it has been created this now luks device can be mounted. 

`cryptsetup` is the tool to set up a luks encryption. if you run `cryptesetup --help` you can see many different actions to take with the tool.

To run a luks encryptions run `cryptsetup luksFormat <Device>`. The prompt will ask for confirmation. Next choose the passphrase. 

TO use the device `cryptsetup luksOpen <Device> <Name-to-use>`. You should be able to see the device mapper directory `/dev/mapper`.

Mounting the device run `mount /dev/mapper/<Name-to-use> /<Location-to-mount>`.
TO close the device , first umount the device `umount /dev/mapper/<Name-to-use>` and then run `cryptsetup luksClose /dev/mapper/<Name-to-use>`. 
TO verify that the device has gone, go to the directory that the device lives in and run and `ls`. 

### Mounting devices persistently

A key will need to be created so you canuse your device persistently when loging on ot your machine. HAving the mounted device on your machine you an make life simpler by having hte key loaded on a removeable USB or similar. this way the mount can persist but the access will not. 

For an automatic mount luks open will need to be automated. The mount will also need to be automated. 


For an example You can create a random key of stuff with the command 
`dd if=/dev/urandom of=/root/luksey bs=4096 count=1`
Give the key root provledges `chmod 600` and then add the key to the encrypted device with `cryptsetup luksAddKey <Device-location> <Key-Location>`. You will be asked to set up a passphrase for the key. 
Next create the file `vim /etc/crypttab`. In this new file provide he name of the device, the underlaying device and then the name of the key. Then modify the `/etc/fstab` to include the <Device-name-and-location> and the location of where yu want to mount it.For example a line like
```
/dev/mapper/MyNewsecureFileSystem          /Desktop    ext4 defaults 1 2 
```
To check if this has worked you will need to reboot the system. 

### Security Related Mount options 

 `nodev`  Do not interpret character or block special devices on the file system.
 `nosuid` Do not allow set-user-ID or set-group-ID bits to take effect.
 `noexec` Do not permit direct execution of any binaries on the mounted filesystem.

 ```
 mount -o remount,noexec <address-of-the-file-system>
 ```

 ## <ins>Securing Server Access</ins>

 ### Secureing the GRUB boot loader

 When you turn the powere on the machine will run a POST (Power On Self Test). IT will then look for a bootable device. Once this is found , it will look for the `Bootloader`, normally `GRUB2`. GRUB2 will load the `Kernel` and the `initramfs`. After the service start we will be able to get a `shell`. 

Although we cannot prevent an attacker putting a physical thumbdrive in o the device, the first place we can protect before the Linux Kernel is loaded is the GRUB2 boot loader. This allows users to put in boot arguments. Controlling these will protect the system.

GRUB2 has two password types
- the global password : Makes it impossible to enter no matter what is on the prompt.
- OS specific password: Secures juts one specific OS started from GRUB.


To protect the password on GRUB2 you will need to make 2 input files with the following input:
- `/etc/grub.d/01_users`
```
set superusers="bob"
password bob somesecretepassword
password alice adifferentsecretpassword
```
- `/etc/grub.d/40_custom` defines verbatim menu entries for the GRUB2 menu, way before the shell.
```
#!/bin/sh
exec tail -n +3 $0
# This file provides an easy way to add custom menu entries.  Simply type the
# menu entries you want to add after this comment.  Be careful not to change
# the 'exec tail' line above.
menuentry 'CentOS Linux (3.10.0-327.13.1.el7.x86_64) 7 (Core)' --users bob {
set root=(hd0, msdos1)
linux16 /vmlinuz-3.10.0-327.13.el7/x86_64
}
```
**These files will need to be written to** `/boot/grub2/grub.cfg`.

In GRUB1 things were much easier. You would just need to specify `password` `<secret>`.

## Manageing Kernel Security

Because the Linux Kernel is based on Unix, which was not concerned with security, neither has Linux been concerned. Security has been built into the Linux Kernel progressively over time. 

Firstly is the ring architecture that Linux is useing. Eg; 
- Ring 0: Kernel
- Ring 3 : User space

Programs are given defined allocations of memory. If they wanted to get more they would have to get the kernel to permit this which would involved a `syscall` process being executed. This syscall would act in the kernel space and return with having made the changes in kernel space. The point her eis that processes are isolated. 

if you do need to make a cross process call you would require an Inter Process Comunication; configured at the kernel level. The alternative is the `mmap` system call which is a specific system call to let one program communicate directly with another program.

Processes have to be isolated from each other as best they can to prevent breaches. Solutions ot this include 
- `chroot`: which will run a process in a fake root environment
- `cgroup`: allows specific hardware to be available for different processes.
- `containers`: Docker, Linux Container Project(LXC)

### Linux Kernel Security Issues

**Buffer overflow** : BReaching the memory allocatedd to a process with a specific command that can be executed. Patching and input validation are the remedy.

**Privledge Escalation:** among other reasons, if the SUID is set on a program it will run as the owner of the program. If the root owns the program, it is pretty bad. Even a bash script. Also Shells from `su` or `sudo`.

**Root kit** : This will contain fake programs such as a fake `ls` or `cat`. As soon as the root user executes on of these fakes, control is handed to the attacker. Root kit prevention is however quite simple.
- Run a file system integrity checker like AIDE.
- no kernel modules

### Kernel modules

You can list the status of kernel modules with the `lsmod` command. 
`modprobe` is a tool to add and remove modules from the Linux Kernel.

Within the file `/proc/sys/kernel/modules_disabled` is kept a single value of 0 for disabled and 1 for enabled. This file sets the permission for allowing Kernel modules t obe installed, IF our system has everything it need this could then be set to 1 by echoing 1 to the file. **WARNING!!!** This may make you system inflexible, especially if you update hardware and the only way to reset it is to reboot your system. It should only be added to a live system and not permenant through sysctl.

The `/proc/sys/kernel/randomize_va_space` contains a setting for memory addres randomization for processes. The value of `2` being the most random. This will make it hard to know how memory is being allocated and there for prevent certain attacks. This is however, "Security thorugh obscurity" and not ideal. 

### no execute value 

Within the `/proc/cpuinfo` file is listed many flags regarding the configuration of the cpu. One of which `nx` which stand for `No execute`. If set will prevent important areas such as the heal and stack on the cpu are protected from executing any code. 

# Manageing Linux permissions and attributes

## Basic Permissions

|  Number      |      Permission Type          |    Symbol |   |   |
|:------------:|:-----------------------------:|:---------:|---|---|
|   0          |     No Permission             |     —     |   |   |
|   1          |     Execute                   |     –x    |   |   |
|   2          |     Write                     |     -w-   |   |   |
|   3          |     Execute + Write           |     -wx   |   |   |
|   4          |     Read                      |      r–   |   |   |
|   5          |     Read + Execute            |      r-x  |   |   |
|   6          |     Read + Write              |     rw-   |   |   |
|   7          |     Read + Write + Execute    |     rwx   |   |   |

To work with permissions is about ownership. Permissions are assigned UGO (Users,Groups, others). `chown` is used for the changeing the owner. `chgrp` is used for changing the group owner. If you don't change ownership then the user who created the file will be the user owner and the primary user of that group will become group owner. 

The permission mode is where you define which permission will me applied to yor files.
`chmod` is used for change the mode of a file. `chmod` can used in an absolute way and a relative way such as `chmod 761`. Lets break that command down

| example UGO permissions | user | group | others |   |
|-------------------------|------|-------|--------|---|
| chmod                   | 7    | 6     | 1      |   |
|                         |      |       |        |   |
|                         |      |       |        |   |
 
 So above we would be assigning 
 - user to `Read + Write + Execute `
 - group to `Read + Write`
 - others to `Execute`

 ```sh
 chown <user>:<Group_name> <Directory>
 chown alice:foo bar/                   # add alice of the foo group to the bar directory
 ```


 ## Special Permissions 
 These were invented to solve a few problems that occurred with the first distributions of Unix, and have been addopted into Linux.  

 |            | files | directories |
|------------|-------|-------------|
| SUID       |  Run file as owner     |   `<n/a>`          |
| SGID       |    Run file as group owner   |   Inherit the directory group owner          |
| Sticky bit |    `<n/a>`  |      only delete if you are owner       | 


Note: SUID is not effective on shell scripts. 


In order to assign special permissions we can use chmod by adding a 4th digit to the left hand side of the regular permission commands for example

| example UGO permissions | special | user | group | others|
|-------------------------|------|-------|--------|---|
| chmod                   | 4 or 2 or 1   | 7    | 6     | 1      |
|                         | 4 (SUID)  | 7    | 6     | 1      |
|                         | 2 (SGID)  | 7    | 6     | 1      |
|                         | 1 (Sticky bit)   | 7    | 6     | 1      |
|                         |      |       |        |   |

You can set the Set User ID with `chmod u+s <file>`

To find files that have the `SUID, SGID or sticky bit` set you can use the following command which will look for permissions that have 4 digits , 3 of which don't matter, just the one on the left which is set to X atm.
      ```
      find / -perm 4000    # files with SUID set, exactly
      find / -perm -4000   # files with SUID and any other permissions
      find / -perm /X000   # fiels with SUID and anything else
      find / -perm 2000    # files with SGID set, exactly
      find / -perm /6000    # files with SGID, SUID and any other permissions. The best search to do
      ```

### Sticky bit

Sticky bit on a file was set on very old Unix versions and meant that the file needed to be kept in cache as long as possible, before 64gb rams and more. It was not applied as an option for files on Linux.**If sticky is applied , you can only delete a file if you are the user owner of the file or , user owner of the directory that contains the file.** 
To set the sticky bit run `chmod +t <file>` or with `chmod 1xxx`
To remove the sticky bit run `chmod -t <file>` or with `chmod 0xxx`

To clarify, if the sticky bit is set on a directory by a user and that directory lives in a directory owned by you, then you can still delete the directory that the other users directory lives in because you own that parent directory.

If the sticky-bit is set on a file or directory without the execution bit set for the others category (non-user-owner and non-group-owner), it is indicated with a capital T (replacing what would otherwise be -) and perhaps blue around the listing of the file name.
```
-rw-r--r--   1 root root 102400 Nov 10 12:57 MyFile.txt
or 
-rw-r--r-T   1 root root 102400 April 1 01:39 MyFile.txt
```



# Iptables Basics

In the Linux Kernel, firewall functionality is implimented with  `netfilter`. Traffic is passed through this to filter it. The main interface to `netfilter` is `iptables`. iptables has a lot of advanced features that gives a lot of precision but this comes at the cost of the complexity of useing it. To simplify things `ufw` (uncomplicated firewall) and `firewalld` are front end to the iptables utilities. ufw is the default on Ubuntu. Firewalld is the default on RedHat and others.

`iptables` works with tables and the default table is the `filter` table. Within a table there are `chains`. Chains decide what kind of packet flows should be fileted. Amongother chains you will have an `INPUT` chain, an `OUTPUT` chain; and if the system is a router you may have a `FORWARD`.

If you are going to do advanced things like ip masquerading, you may use `prerouting` and `postrouting` chains. These two are essential if you want to do NAT (`network address translation`) as is used in the `mangle` table where the ip address is changed. 

Within he chain the next element that is going to be changed are the `rules`. These define what should happen to a packet. Rules use a principle of `exit on match` where by if all items of the rule a met the rule will be applied and nothing will be done in that chain any more. This therfore makes ordering very important. 

in eey rule there is also a `target` these are specified useing the `-j` option. Typical targets include:
- ACCEPT - to allow
- DROP - Silently dropped
- REJECT - dropped with a ICMP warning message

In any chain there is a `Policy` which defines the default behavior. Its good practice to have a default policy that drops anything that doesn't match a specific packet in a chain. 

### Default Components of an iptables command

`iptables -A <CHAIN_NAME> <INTERFACE> <ADDRESS> -p <PORT> -j <TARGET>`

- -A : append to the end
- `<CHAIN_NAME>`: input output, forward etc
- `<INTERFACE>` -i or -o for input or output interface and then the name of the interface
- `<ADDRESS>` -s `<ip_address>` -d `<ip_address>` for source or destingation addresses. either can be filtered
- -p : set the protocol eg tcp,udp,ssh
- `<PORT>` : --sport 80 or --dport 80 for sourceport 80 or destination port 80
- -j `<TARGET>` define what will happen to the packet that will match the rule.



### Modifying text Console Settings

The login process uses 

- `/etc/issue `- this is a file that will display before login. It might have some config in the first few lines.
- `/etc/motd` - after login there is  the `message of the day`.
- `/.hushlogin` in the home of a user, contents of the motd will ot be shown

### <ins>Modifying Graphical Console Settings<ins/>

`cd /etc/gconf` is where the configuration will be kept
`gconftool-2 --help` has lots of sub help catagories 


# <ins> Securing Linux Infrastructure </ins>

### Sniffing and port scanning 

In modern networks we prefer `Switches` over `hubs`. This is because they have a MAC address table so the packets will only be advertised to the target port and none other.

 TO use tcpdump you need to understand your network cards. To do this run the command `ip link show`.

 **tcpdump** needs to be running as root because only rot can capture packets from the network.
 the manadory argument is hte network card you want to be listening on there fore you would run something like the following `tcpdump -i eth0`. The out put will contain things that are structure as 

 `<TIMESTAMP>  <PROTOCOL> <INVOLVED MACHINE INFORAMTION>`

 ```
 07:29:26.769165 STP 802.1w, Rapid STP, Flags [Learn, Forward], bridge-id 89c5.70:70:8b:20:77:0f.80d9, length 42
07:29:26.777079 ARP, Request who-has 10.51.9.64 tell 10.51.0.1, length 42
07:29:26.798437 ARP, Request who-has 10.51.11.85 tell 10.51.11.84, length 42
```

**tcpdump** and wire shark bot handle `packet capture` files aka `.pcap` files. 

`tcpdump -i eth0 -w $(date +%d-%m-%Y).pcap` This will write the output of the tcpdump to a pcap file with the date in the title of the file.


`tcpdump -n -i eth0` will not show the host names and instead p addresses
`tcpdump -w ssh.pcap -i eth0 dst 192.168.4.10 and port 22` # filter on particular ips and ports , and write t oa certain file

```sh
nmap -sn <Network_IP+SubnetMask>        # scans the entire network
nmap -v -A <Network_IP+SubnetMask>      # agressive verbose network scan
nmap -PN <Ip_Address>                   # pierces through the fire wall apparently ???
nmap -O <Ip_Address>                    # scans for the operating system
nmap -PA                                # tcp AK scan ( 2nd half handshake )
```

### Tripwire

Tripwire has two versions open source and commercial
- open source version: HIDS ( Host Intrusion Detection System)
- Commercial version: NIDS ( Network Intrusion Detection System)

NIDS is more complicated that works with profiles that are part of the paid updates. An alternative is Snort but this also has a cost. An alternative to the opensource versions Aide is a good alternative. 


# Configureing Linux Logs

There is no standardizations in Linux Logging solutions. 
Logging is done by `Services`, `Syslog`, `Systmed-journald` and if everything is going ok these will all be sent to the `/var/log`.

syslog is loggin by using `facilities` such as `news` , `cron` , `kern` etc. This was based around a previous time when there were not so many services as there are now and so now services have started to make there own logging method. 

Syslog has become the defacto standard logging. The syslog service used facilities, priorities and sending it on to a destination. As more service s have been added syslog need a more advanced log service was needes, enter `syslog-ng`(next generation). 

`rsyslog` has replace syslog-ng as this can work with modules to make the logging more flexible. IT provides complete backward compatability and privides modules for extra features, for example for handling IO/OP directions.

The main place to configure rsyslog is the `/etc/rsyslog.conf`. `im` is for input modules and `om` is for output modules. 

`*.* @@remote-host:514` means every facility and every priority is sent via tcp to remote host on poirt 514.

journald is a volatile logging message platform that you can browse through the messages using the command `journalctl`. 

## Configureing remote logging

Today, secure remote logging will use `tls`. 
Requirements will include
- time sync . without this it will be pointless
- tls certs need setting up. (for now cirttool will sort this)
- tcp port 6514 neeeds to be accessible for the log server. 
- GTLS driver needs to be configured.

It is recommended to install the `rsyslog-doc` for all the documentation.

As part of the RHEL exam, it will involve getting the `native tls encryptions for syslog`. 

1. `systemctl status chronyd` - Check this time service on RHEL7+ ( else NPTD)
2. if `certtool` is not installed run `yum install gnutls-utils` to get it. 
3. Run `certtool --generate-privkey --outfile ca-key.pem `
4. give the ca-key.pem readable permission with `chmod 400 ca-key.pem`
5. Create a public key from the private key you have just made with `certtool --generate-self-signed --load-privkey ca-key.pem --outfile ca.pem`
6. Choose the settings you would like for your cert includeing.
    - Does the certificate belong to an authority? y
    - set no constraint on the path length.
    - the dnsName of the subject will be the will be the server name.
    - the URI,Ip and email of the subject do not apply.
    - No to: sign code, OCSP requests, time stamping.

7. next to genreate a private key.  
`certtool --genrate-privkey --outfile server1-key.pem --bits 2048`
8. Generate a signer request to get the CA.
`certtool --generate-request --load-privkey server-key.pem --outfile server1-request.pem`.
9. After chooseing all the options from the above command it will generate the pem files for the server. To make the key materiel for the rsyslog client `certtool --generate-certificate --load-request server1-request.pem --outfile server-cert.pem --load-ca-certificate ca.pem --load-ca-privkey ca-key.pem` . This will make sure that server1 is trusted by all involved. 

### Setting up the log server
once the certs are created for hte CA and the server.
1. On server1 make a dir `/etc/rsyslog-keys`
2. Using `scp` (or similar) , copy the keys over to the server1 dir `scp server*.pem <IP_Addr_OF_MAchine>:/etc/rsyslog-keys` 
3. Add the ip addresses and dns names of workstation and server to your `/etc/hosts` file.
4. Configure the syslog on server1 for reception on port 6514 by going to `/etc/rsyslog.d` and adding 
```sh
$DefaultNetstreamDriver gtls
$DefaultNetstreamDriverCAFile /etc/rsyslog-keys/ca.pem
$DefaultNetstreamDriverCAFile /etc/rsyslog-keys/server1-cert.pem
$DefaultNetstreamDriverCAFile /etc/rsyslog-keys/server1-key.pem

$Modload imtcp

$InputTCPServerStreamDriverMode 1
$InputTCPServerStreamDriverAuthMode anon
$InputTCPServerRun 6514
```
5. restart the syslog service. You may need to install some packages if this complains and run again.
6. Configureing the client requires a directory `mkdir /etc/rsyslog-keys`.
7. Copy the `ca.pem` into there. 
8. Create a configuration file. The name is not as important as the content. IT should inlcude 
```sh
$DefaultNetStreamDriverCAFile /etc/rsyslog-keys/ca.pem

$DefaultNetStreamDriver gtls            # this package may need to be installed. Maybe "gnutls: 
$ActionSendStreamDriverMode 1           # this requires rsyslog to use tls
$ActionSendStreamDriverAuthMode anon    
*.*     @@(o)server1.example.com:6514   
```
9. install any packages that are needed for this and then restart the rsyslog server with systmectl.

### Manageing Log rotation

The solution to have a long range rotation so you can have a larger log window going back months or years is to compress logs after some time and store then off your device.

Look in the file `/etc/cron.daily/logrotate`. This will probably be referenceing the `/etc/logrotate.conf`. these will contain the configuration options for the log rotation. There is also `logrotate.d` for other files. In addition logrotate will incorporate the default parameters from the different logging engines, for example the settings in syslog.

### journald persistence

`journald` is part of systmed and may eventually replace syslog. For example Linux SUZE has not syslog as default.  journald send its logs to the `/run/log/journal` file. There is a parameter for configuring the rotation file size. Some Linux distros currently have only summary logs from journald go to syslog; just enough for an admin but no more. 
To get persistence with more comprehensive journald logs just make a certain directory and then restart the service with
```sh
 mkdir -p /var/log/journal
 systemctl restart systemd-journald
 ```
Just be carful for the rotation size. 
The takeaway here is that rsyslog offerers remote logging where as journald doesn't so, try and hold on to rsyslog as long as you can. 

### Useing logwatch for log analysis

One of the solutions for log analysis is logwatch. logwatch runs from a cronjob. OS you can see it in the cron.daily. Configurations setting are set in a `logwatch.conf` file. You can run an up to date log with `logwatch --range all` which will include todays messages. 