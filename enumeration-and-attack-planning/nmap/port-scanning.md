---
description: '-Pn  : Avoid host discovery phase.'
---

# Port Scanning

## Default scans

Nmap offers a lot of scanning options to specify the scanning technique. But if none were specified, nmap chooses a default scan type which depends solely on the privileges the user have. Here's a quick explanation:

<img src="../../.gitbook/assets/file.excalidraw (12).svg" alt="Figure1" class="gitbook-drawing">

## Discovering OPEN TCP Ports

```bash
└──╼ $nmap -Pn -n 10.10.10.10
```

## Discovering OPEN UDP Ports (Much slower than TCP)

```bash
└──╼ $nmap -Pn -n -sU 10.10.10.10
```

{% hint style="danger" %}
* We do not receive any acknowledgment. Consequently, <mark style="color:red;">**the timeout is much longer**</mark>, making the whole `UDP scan` (`-sU`) much slower than the `TCP scan` (`-sS`).
* Also We often do not get a response back because `Nmap` sends empty datagrams to the scanned UDP ports, and we do not receive any response. So we cannot determine if the UDP packet has arrived at all or not. <mark style="color:red;">**If the UDP port is**</mark><mark style="color:red;">** **</mark><mark style="color:red;">**`open`**</mark><mark style="color:red;">**, we only get a response if the application is configured to do so.**</mark>
{% endhint %}

<img src="../../.gitbook/assets/file.excalidraw (8).svg" alt="UDP scans possible outputs." class="gitbook-drawing">

## Service detection:

```bash
└──╼ $sudo nmap 10.10.10.10 -p- -sV

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 20:00 CEST
Nmap scan report for 10.129.2.28
Host is up (0.013s latency).
Not shown: 65525 closed ports
PORT      STATE    SERVICE      VERSION
22/tcp    open     ssh          OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
25/tcp    open     smtp         Postfix smtpd
80/tcp    open     http         Apache httpd 2.4.29 ((Ubuntu))
110/tcp   open     pop3         Dovecot pop3d
139/tcp   filtered netbios-ssn
143/tcp   open     imap         Dovecot imapd (Ubuntu)
445/tcp   filtered microsoft-ds
993/tcp   open     ssl/imap     Dovecot imapd (Ubuntu)
995/tcp   open     ssl/pop3     Dovecot pop3d
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)
Service Info: Host:  inlane; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 91.73 seconds
```

Primarily nmap <mark style="color:blue;">**prints out the banners of the scanned ports**</mark>. If nmap cannot identify versions through banner grabbing, it performs a signature-based matching system which increases the duration.

{% hint style="danger" %}
Nmap can miss some information because <mark style="color:red;">**sometimes it doesn't know how to handle some information.**</mark>

It happens because, after a successful three-way handshake, the server often sends a banner for identification. This serves to let the client know which service it is working with. At the network level, this happens with a `PSH` flag in the TCP header. This mechanism is described in the [<mark style="color:red;">first figure.</mark>](port-scanning.md#id-0.-default-scans)
{% endhint %}

### Banner grabbing for service detection:

<pre class="language-bash"><code class="lang-bash">└──╼ $ sudo nmap 10.10.10.10 -p- -sV -Pn -n --disable-arp-ping --packet-trace

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-16 20:10 CEST
&#x3C;SNIP>
NSOCK INFO [0.4200s] nsock_trace_handler_callback() .. SNIP .. ESMTP 
Postfix (Ubuntu)..
It detected that it is Ubuntu

Service scan match (Probe NULL matched with NULL line 3104): 10.129.2.28:25 is smtp.  Version: |Postfix smtpd|||
NSOCK INFO [0.4200s] nsock_iod_delete(): nsock_iod_delete (IOD #1)
Nmap scan report for 10.129.2.28
Host is up (0.076s latency).

PORT   STATE SERVICE VERSION
25/tcp open  smtp    <a data-footnote-ref href="#user-content-fn-1">Postfix smtpd</a>
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)
Service Info: Host:  inlane

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 0.47 seconds
</code></pre>

Let's prepare the intercept network traffic using **`tcpdump`**

```bash
└──╼ $ sudo tcpdump -i eth0 host 10.10.10.10 and 1.1.1.1
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
```

Now let's netcat to the target machine an look at the intercepted traffic in tcpdump output:

```bash
└──╼ $ nc -nv 10.10.10.10 25
Connection to 10.10.10.10 port 25 [tcp/*] succeeded!
220 inlane ESMTP Postfix (Ubuntu)
```

```bash
18:28:07.128564 IP 1.1.1.1.59618 > 10.10.10.10.smtp: Flags [S], seq 1798872233, win 65535, options [mss 1460,nop,wscale 6,nop,nop,TS val 331260178 ecr 0,sackOK,eol], length 0
18:28:07.255151 IP 10.10.10.10.smtp > 1.1.1.1.59618: Flags [S.], seq 1130574379, ack 1798872234, win 65160, options [mss 1460,sackOK,TS val 1800383922 ecr 331260178,nop,wscale 7], length 0
18:28:07.255281 IP 1.1.1.1.59618 > 10.10.10.10.smtp: Flags [.], ack 1, win 2058, options [nop,nop,TS val 331260304 ecr 1800383922], length 0
18:28:07.319306 IP 10.10.10.10.smtp > 1.1.1.1.59618: Flags [P.], seq 1:36, ack 1, win 510, options [nop,nop,TS val 1800383985 ecr 331260304], length 35: SMTP: 220 inlane ESMTP Postfix (Ubuntu)
18:28:07.319426 IP 1.1.1.1.59618 > 10.10.10.10.smtp: Flags [.], ack 36, win 2058, options [nop,nop,TS val 331260368 ecr 1800383985], length 0
```

{% hint style="info" %}
1. SYN sent.
2. SYN/ACK received.
3. ACK sent.
4. <mark style="color:blue;">PSH/ACK received. (Exactly where the confusion happens.)</mark>
5. ACK sent.
{% endhint %}



[^1]: But here, not enough information\
    &#x20;shown as supposed to be.
