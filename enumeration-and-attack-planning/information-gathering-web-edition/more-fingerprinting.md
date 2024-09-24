# More Fingerprinting

## Firewall detection

It's common for web apps to contain Web Application Firewalls (WAF) so let's proceed by trying to detect if any exist:

```shell-session
$ pip3 install git+https://github.com/EnableSecurity/wafw00f
$ wafw00f <DOMAIN>
```

## Nikto

Nikto is a large vulnerability assessment tool that has useful fingerprinting modules.

```shell-session
$ sudo apt update && sudo apt install -y perl
$ nikto -h <DOMAIN> -Tuning b
The -h flag specifies the target host.
The -Tuning b flag tells Nikto to only run the Software Identification modules.
```
