# Active Enumeration

## Bruteforcing

We can use : dnsenum, fierce, dnsrecon, amass, assetfinder, puredns. Of course each tool has its strengths. We'll be using dnsenum since it is versatile and provide various functionalities:

```shell-session
$ dnsenum --enum inlanefreight.com -f /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -r
-r : Enable Recursive subdomain bruteforcing.
```
