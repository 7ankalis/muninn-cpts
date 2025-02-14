---
description: A bit "detailed" description and terminology we'll encounter along the way
---

# Password Attacks

## Linux

### Passwd&#x20;

In the past the encrypted password was stored along with the username in the <mark style="color:purple;">**/etc/passwd**</mark>, which was a security issue due to its availability for all users in the following format:&#x20;

&#x20;           <mark style="color:purple;">**\<username>:\<password>:uid:gid:,,,:\<home\_dir>:\<cmd\_login>**</mark>

<figure><img src="../../.gitbook/assets/image (63).png" alt=""><figcaption></figcaption></figure>

In here x means that the password is put in the /etc/shadow file.

For further reading about the Linux[ authentication mechanism](https://tldp.org/HOWTO/pdf/User-Authentication-HOWTO.pdf).

### Shadow

The /etc/shadow file contains the hashed passwords of the users on the target machine in a specific format:&#x20;

<figure><img src="../../.gitbook/assets/image (60).png" alt=""><figcaption></figcaption></figure>

In the format:&#x20;

<figure><img src="../../.gitbook/assets/image (62).png" alt=""><figcaption></figcaption></figure>

The hashed password itself comes in a specific format:

<figure><img src="../../.gitbook/assets/image (64).png" alt=""><figcaption></figcaption></figure>

## Windows

The windows authentication process is much more complicated than the Linux one, it has multiple and various modules that are responsible of the logon, retrieval and verification processes. Here's [more ](https://learn.microsoft.com/en-us/windows-server/security/windows-authentication/credentials-processes-in-windows-authentication)about it.

<figure><img src="../../.gitbook/assets/image (67).png" alt=""><figcaption></figcaption></figure>

### LSA (Subsystem)- Local Security Authority&#x20;

It is a protected subsystem that authenticates users and logs them into the local computer. It maintains information about the aspects of the local security and provides multiple services for translating between names and SIDs (Security IDs).

It keeps track of the security policies and accounts that reside on a computer system along with providing services for checking access to objects, user permissions ..etc

### LSAAS (Service) - Local Security Authority Subsystem Service

LSAAS is a collection of many modules and has access to all authentication processes that can be found in _<mark style="color:purple;">**%SystemRoot%\System32\Lsass.exe**</mark>_. This service is responsible for the local system security policy, user authentication, and sending security audit logs to the <mark style="color:purple;">**Event log**</mark>. In other words, it is <mark style="color:red;">**the vault**</mark> for Windows-based operating systems, and we can find a more detailed illustration of the LSASS architecture [here](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-2000-server/cc961760\(v=technet.10\)?redirectedfrom=MSDN).

<figure><img src="../../.gitbook/assets/image (65).png" alt=""><figcaption></figcaption></figure>

| **Authentication Packages** | **Description**                                                                                                                                                                                                                                                |
| --------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Lsasrv.dll**              | The LSA Server service both enforces security policies and acts as the security package manager for the LSA. The LSA contains the Negotiate function, which selects either the NTLM or Kerberos protocol after determining which protocol is to be successful. |
| **Msv1\_0.dll**             | Authentication package for local machine logons that don't require custom authentication.                                                                                                                                                                      |
| **Samsrv.dll**              | The Security Accounts Manager (SAM) stores local security accounts, enforces locally stored policies, and supports APIs.                                                                                                                                       |
| **Kerberos.dll**            | Security package loaded by the LSA for Kerberos-based authentication on a machine.                                                                                                                                                                             |
| **Netlogon.dll**            | Network-based logon service.                                                                                                                                                                                                                                   |
| **Ntdsa.dll**               | This library is used to create new records and folders in the Windows registry.                                                                                                                                                                                |

### SAM (Database) - Security Account Manager&#x20;

{% hint style="info" %}
<mark style="color:red;">This will be a target.</mark>
{% endhint %}

The Security Account Manager (SAM) is a database file in Windows operating systems responsible for storing user passwords. It facilitates authentication for both local and remote users while employing cryptographic protections to prevent unauthorized access. Passwords are stored as hashes in a registry structure, using either the LM or NTLM hash format. The SAM file is located at <mark style="color:purple;">**%SystemRoot%/system32/config/SAM**</mark> and is mounted under `HKLM/SAM`, requiring SYSTEM-level permissions for access.

During setup, Windows systems can be configured as part of either a workgroup or a domain. In a workgroup setup, the SAM database is managed locally, storing all user accounts on the machine itself. However, in a domain environment, authentication is handled by the Domain Controller (DC), which verifies credentials against the Active Directory database (<mark style="color:purple;">**ntds.dit**</mark>) stored at <mark style="color:purple;">**%SystemRoot%\ntds.dit**</mark>.

To enhance the security of the SAM database against offline cracking attempts, Microsoft introduced the SYSKEY (<mark style="color:purple;">**syskey.exe**</mark>) feature in Windows NT 4.0. When enabled, it encrypts the password hash values stored in the SAM using a separate key, adding an extra layer of protection against unauthorized access.

#### Credential Manager

Credential Manager is a feature built-in to all Windows operating systems that allows users to save the credentials they use to access various network resources and websites. Saved credentials are stored based on user profiles in each user's <mark style="color:purple;">**Credential Locker**</mark>. Credentials are encrypted and stored at the following location:

```powershell
PS C:\Users\[Username]\AppData\Local\Microsoft\[Vault/Credentials]\
```

## NTDS (file)

{% hint style="info" %}
<mark style="color:red;">This will be a target.</mark>
{% endhint %}

In the case of centralized management, each Domain Controller has a file called NTDS.dit file which is a database file that stores the data in Active Directory that is being synchronized across all Domain Controllers (with the exception of Read-Only DC).&#x20;

The data stored could be:

* User Accounts (usernames and hashed passwords)
* Group Accounts
* Computer Accounts
* Group Policy Objects
