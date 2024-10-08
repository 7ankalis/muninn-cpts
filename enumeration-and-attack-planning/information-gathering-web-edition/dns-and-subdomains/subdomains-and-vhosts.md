# Subdomains & VHosts

## Subdomains

It is important for us as Pentesters to find and explore any hidden parts of the target's domain. That being said, subdomains represent a source of valuable information since they can hold hidden login portals, legacy apps, development and staging environments ...etc So there's a high chance we find a foothold through them.

We can find these subdomains with two approaches:

1. Active Enumeration: Brute forcing using wordlists of potential subdomain names against the target system.
2. Passive Enumeration: Look into Certificate Transparency logs, SSL/TLS certificates ..etc

## Virtual Hosting

In short, it's the ability of hosting multiple websites and domains in one server without the need of another web server to host other websites.

This raises a confusion between Virtual Hosts and Subdomains. Here's the difference:

* <mark style="color:blue;">Subdomains:</mark> These are extensions of a main domain name (e.g., `blog.example.com` is a subdomain of `example.com`). `Subdomains` typically have their own `DNS records`, pointing to either the same IP address as the main domain or a different one. They can be used to organize different sections or services of a website.
* <mark style="color:blue;">Virtual Hosts (VHosts):</mark> Virtual hosts are configurations within a web server that allow multiple websites or applications to be hosted on a single server. They can be associated with top-level domains (e.g., `example.com`) or subdomains (e.g., `dev.example.com`). Each virtual host can have its own separate configuration, enabling precise control over how requests are handled.

So, we now know that vhosts can also be configured to use different domains not just subdomains:

```apacheconf
# Example of name-based virtual host configuration in Apache
<VirtualHost *:80>
    ServerName www.example1.com
    DocumentRoot /var/www/example1
</VirtualHost>

<VirtualHost *:80>
    ServerName www.example2.org
    DocumentRoot /var/www/example2
</VirtualHost>

<VirtualHost *:80>
    ServerName www.another-example.net
    DocumentRoot /var/www/another-example
</VirtualHost>
```

## Fuzzing

{% code overflow="wrap" %}
```shell-session
$ gobuster vhost -u http://<target_IP_address> -w <wordlist_file> --append-domain
    Consider using the -t flag to increase the number of threads for faster scanning.
    The -k flag can ignore SSL/TLS certificate errors.
    You can use the -o flag to save the output to a file for later analysis.

$ ffuf -w <WORDLIST>:FUZZ -u http://FUZZ.TARGET:PORT/ -c -v 
$ ffuf -w <WORDLIST>:FUZZ https://target:port/ -H 'Host: FUZZ.target' -c -v 
    Consider other filter and matching options that ffuf offers (-fw,-fc,-mw...)

$ dnsenum --enum <TARGEt> -f <WORDLIST> -r
-r : Enable Recursive subdomain bruteforcing.
```
{% endcode %}

{% hint style="warning" %}
In newer versions of Gobuster, the <mark style="color:blue;">--append-domain</mark> flag is required to append the base domain to each word in the wordlist when performing virtual host discovery. This flag ensures that Gobuster correctly constructs the full virtual hostnames, which is essential for the accurate enumeration of potential subdomains. In older versions of Gobuster, this functionality was handled differently, and the <mark style="color:blue;">--append-domain flag</mark> was not necessary. Users of older versions might not find this flag available or needed, as the tool appended the base domain by default or employed a different mechanism for virtual host generation.
{% endhint %}
