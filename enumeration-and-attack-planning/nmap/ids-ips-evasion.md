---
description: Stalking isn't that easy.
---

# IDS/IPS Evasion

## Firewall Detection

### Testing Firewall rules  (Using ACK scan)

{% hint style="warning" %}
<mark style="color:orange;">When sending an ACK flag, BOTH Open and Closed ports will reply with an RST flag!</mark>
{% endhint %}

<pre class="language-bash"><code class="lang-bash">└──╼ $ sudo nmap 10.10.10.10 -p 21,22,25 -sS -Pn -n --disable-arp-ping --packet-trace

SENT (0.0278s) TCP 10.10.14.2:57347 > 10.10.2.10:21 S .. &#x3C;SNIP> ..
SENT (0.0278s) TCP 10.10.14.2:57347 > 10.10.10.10:22 S .. &#x3C;SNIP> ..
SENT (0.0278s) TCP 10.10.14.2:57347 > 10.10.10.10:25 S .. &#x3C;SNIP> ..

RCVD (0.0329s) ICMP [10.10.10.10 > 10.10.14.2 Port 21 unreachable (type=3/code=3) ] .. &#x3C;SNIP> ..
RCVD (0.0341s) TCP 10.10.10.10:22 > 10.10.14.2:57347 SA .. &#x3C;SNIP> ..
RCVD (1.0386s) TCP 10.10.10.10:22 > 10.10.14.2:57347 SA .. &#x3C;SNIP> ..

<a data-footnote-ref href="#user-content-fn-1">SENT (1.1366s) TCP 10.10.14.2:57348 > 10.10.10.10:25 S .. &#x3C;SNIP> ..</a>

PORT   STATE    SERVICE
<a data-footnote-ref href="#user-content-fn-2">21/tcp filtered ftp</a> 
<a data-footnote-ref href="#user-content-fn-3">22/tcp open     ssh</a> 
<a data-footnote-ref href="#user-content-fn-4">25/tcp filtered smtp</a> 
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)
</code></pre>

<pre class="language-bash"><code class="lang-bash">└──╼ $ sudo nmap 10.10.10.10 -p 21,22,25 -sA -Pn -n --disable-arp-ping --packet-trace

SENT (0.0422s) TCP 10.10.14.2:49343 > 10.129.2.28:21 A .. &#x3C;SNIP> ..
SENT (0.0423s) TCP 10.10.14.2:49343 > 10.129.2.28:22 A .. &#x3C;SNIP> ..
SENT (0.0423s) TCP 10.10.14.2:49343 > 10.129.2.28:25 A .. &#x3C;SNIP> ..

RCVD (0.1252s) ICMP [10.129.2.28 > 10.10.14.2 Port 21 unreachable (type=3/code=3) ] .. &#x3C;SNIP> ..
RCVD (0.1268s) TCP 10.129.2.28:22 > 10.10.14.2:49343 R .. &#x3C;SNIP> ..

SENT (1.3837s) TCP 10.10.14.2:49344 > 10.129.2.28:25 A .. &#x3C;SNIP> ..

PORT   STATE      SERVICE
<a data-footnote-ref href="#user-content-fn-5">21/tcp filtered   ftp</a>  
<a data-footnote-ref href="#user-content-fn-6">22/tcp unfiltered ssh</a> 
<a data-footnote-ref href="#user-content-fn-7">25/tcp filtered   smtp</a>
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)
</code></pre>

## IDS/IPS Evasion

{% hint style="info" %}
We can determine whether there are IDS systems by scanning from a single host from a VPS  and see whether we get banned from that IP or not.
{% endhint %}

Since we must be as quiet as possible in our scans, here we provide some techniques to evade getting caught in our scans:

### Scenario 1 : Hiding our IP with legitimate IPs

We can hide our real IP address in a collection of IP addresses so that the IDS/IPS won't filter out our probes and treat them like legitimate ones.

{% hint style="danger" %}
<mark style="color:red;">ALL the IP addresses we provide must be ALIVE, otherwise it's easy for the monitoring system to detect fake ones and ban us. \\</mark>

So, we can use our VPS IP addresses and use them with "IP ID" headers manipulation.
{% endhint %}

#### Solution : Decoys

The idea is generating random IP addresses and hide our IP in a random position in between.

<pre class="language-bash"><code class="lang-bash">└──╼ $ sudo nmap 10.10.10.10 -p 80 -sS -Pn -n --disable-arp-ping --packet-trace -D RND:5


SENT (0.0378s) TCP 102.52.161.59:59289 > 10.129.2.28:80 S 
<a data-footnote-ref href="#user-content-fn-8">SENT (0.0378s) TCP 10.10.14.2:59289 > 10.129.2.28:80 S</a> 
SENT (0.0379s) TCP 210.120.38.29:59289 > 10.10.10.10:80 S 
SENT (0.0379s) TCP 191.6.64.171:59289 > 10.10.10.10:80 S 
SENT (0.0379s) TCP 184.178.194.209:59289 > 10.10.10.10:80 S 
SENT (0.0379s) TCP 43.21.121.33:59289 > 10.10.10.10:80 S 

RCVD (0.1370s) TCP 10.129.2.28:80 > 10.10.14.2:59289 SA 

PORT   STATE SERVICE
80/tcp open  http
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)
</code></pre>

### Scenario 2 : Some services are only available for specific subnets

#### Solution : Changing source IP address

```bash
└──╼ $ sudo nmap 10.10.10.10 -n -Pn -p 445 -O -S 10.129.2.200 -e tun0

-S 10.129.2.200 : Source of the requests to be sent.
-e : Send all requests through the specified interface.
```

### Scenario 3 : We need to perform DNS resolution but there are DNS-based Firewalls or filtering systems

#### Solution : DNS Proxying

{% hint style="info" %}
The idea is :&#x20;

Instead of sending DNS queries directly to the target's DNS server, the queries are sent to a third-party DNS server (the proxy) or even use the company's DNS server.
{% endhint %}

### Scenario 4 : Hiding our probes with legitimate DNS resolution queries

{% hint style="info" %}
Many DNS requests to be made via <mark style="color:red;">**TCP port 53**</mark> due to changes in DNS resolution methods, so it's better we try to exploit this port.
{% endhint %}

* SYN scan from default port:

```bash
└──╼ $ sudo nmap 10.10.10.10 -p50000 -sS -Pn -n --disable-arp-ping --packet-trace

PORT      STATE    SERVICE
50000/tcp filtered ibm-db2
```

* SYN scan from port 53:

<pre class="language-bash"><code class="lang-bash">└──╼ $ sudo nmap 10.10.10.10 -p50000 -sS -Pn -n --disable-arp-ping --packet-trace --source-port 53

PORT      STATE SERVICE
<a data-footnote-ref href="#user-content-fn-9">50000/tcp open  ibm-db2</a>
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)
</code></pre>

```bash
└──╼ $ ncat -nv --source-port 53 10.10.10.10 50000
Ncat: Version 7.80 ( https://nmap.org/ncat )
Ncat: Connected to 10.129.2.28:50000.
220 ProFTPd
```

{% hint style="success" %}
<mark style="color:green;">We successfully got a connection through the port that was showed as filtered at first.</mark>
{% endhint %}



[^1]: This is sent because by default, when no response is received from a port, Nmap resends another packet just to make sure. (--max-retries=1)

[^2]: ICMP error type 3

[^3]: Received a SYN/ACK flag.

[^4]: Received no response.

[^5]: Received ICMP error code 3

[^6]: Received RST flag=either OPEN or Closed

[^7]: Received no response.

[^8]: Our real IP address

[^9]: It worked as if it's a legit DNS resolution query.
