# Domain Information

## Online Presence:

* [x] SSL certificates.
* [x] IOT devices.
* [x] DNS records.

{% embed url="https://crt.sh/" %}
Certificate Transparency logs
{% endembed %}

## Download the SSL certificates logs:

```bash
nidhÖg@htb[/htb]$ curl -s https://crt.sh/\?q\=domainname.com\&output\=json | jq .
```

### Filtering the output:

#### Filter by unique subdomains:

```bash
nidhÖg@htb[/htb]$ curl -s https://crt.sh/\?q\=domainname.com\&output\=json | jq . | grep name | cut -d":" -f2 | grep -v "CN=" | cut -d'"' -f2 | awk '{gsub(/\\n/,"\n");}1;' | sort -u
```

#### FFilter by company-hosted servers:

```bash
$ for i in $(cat subdomainlist);do host $i | grep "has address" | grep inlanefreight.com | cut -d" " -f1,4;done
```

## Finding IOT Devices:

```bash
nidhÖg@htb[/htb]$ for i in $(cat subdomainlist);do host $i | grep "has address" | grep inlanefreight.com | cut -d" " -f4 >> ip-addresses.txt;done
nidhÖg@htb[/htb]$ for i in $(cat ip-addresses.txt);do shodan host $i;done
```

## DNS records:

```bash
nidhÖg@htb[/htb]$ dig any inlanefreight.com
```

{% hint style="info" %}
* _<mark style="color:blue;">**`A`**</mark>_<mark style="color:blue;">Records:</mark> `Maps a domain name to IPV4 Address:`\
  &#x20;          **`example.com IN A 192.0.2.1`**
* _<mark style="color:blue;">**`PTR`**</mark>_<mark style="color:blue;">Records:</mark> `Maps an IPV4 Address to a domain name:`\
  `           `**`1.2.3.4.in-addr.arpa. IN PTR example.com`**
* _<mark style="color:blue;">**`MX`**</mark>_ <mark style="color:blue;"></mark><mark style="color:blue;">Records:</mark> Points to the mail server responsible for forwarding mail on behalf of a domain. _**(Mail Exchange)**_\
  &#x20;                          **`example.com. IN MX 10 mail.example.com.`**
* _<mark style="color:blue;">**NS**</mark>_ <mark style="color:blue;"></mark><mark style="color:blue;">Records:</mark> responsible for providing DNS information for the domain "example.com".\
  &#x20;                           **`example.com IN NS ns1.example.com`**
* _<mark style="color:blue;">**TXT**</mark>_<mark style="color:blue;">** **</mark><mark style="color:blue;">**`Records:`**</mark>**` ``often`** contains verification keys for different third-party providers and other security aspects of DNS.\
  &#x20;          **`example.com IN TXT "v=spf1 mx -all"`**
{% endhint %}
