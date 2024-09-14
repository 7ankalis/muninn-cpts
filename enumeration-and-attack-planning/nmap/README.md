---
description: >-
  This page is about some basic syntaxes of the input, output and optimization
  options.
---

# Nmap

## Input Options:

```bash
$nmap 10.10.10.10 : Scan a single host
      10.10.10.10-20 : Scan hosts from 10.10.10.10 to 10.10.10.20
      10.10.10.10/24 : Scan the whole subnet.
      -iL <filename> : scan hosts located in the file.
      domain.tld : scan a domain.
```

## Scanning Techniques:

| Switch            | Description                                                                                                      |
| ----------------- | ---------------------------------------------------------------------------------------------------------------- |
| `$nmap -sS  <IP>` | **TCP SYN scan.**                                                                                                |
| `$nmap -sT  <IP>` | <p><strong>TCP Connect scan</strong><br> (By default, this is executed when an unprivileged user runs nmap).</p> |
| `$nmap -sA  <IP>` | **TCP ACK** scan.                                                                                                |
| `$nmap -sU  <IP>` | **UDP** port scan.                                                                                               |

{% hint style="info" %}
These flags (SYN, ACK..etc) are part of the TCP header, which is included in every TCP segment exchanged between devices during the communication process. By examining the flags in the TCP header, devices can determine the purpose of incoming packets and take appropriate actions, such as establishing a connection, acknowledging data receipt, or closing the connection.
{% endhint %}

## Output Options:

| Switch                      |                                                                                                                                                                                                          |
| --------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `$nmap <IP> -oN <filename>` | Save the results in the same format nmap shows the result. (.nmap extension)                                                                                                                             |
| `$nmap <IP> -oG <filename>` | Save the results in Grepable output. (.gnmap extension)                                                                                                                                                  |
| `$nmap <IP> -oX <filename>` | <p>Save the results in XML format. We can later easily crate HTML file using:</p><pre class="language-shell-session"><code class="lang-shell-session">$ xsltproc target.xml -o target.html
</code></pre> |
| `$nmap <IP> -oA <filename>` | Save the results in all the previous formats.                                                                                                                                                            |
| `$nmap <IP> -reason`        | Shows the reason why a speciffic port is shown in a specific state.                                                                                                                                      |
| `$nmap <IP> -open`          | Only show open ports.                                                                                                                                                                                    |
| `$nmap <IP> -packet-trace`  | Show all packets sent and received.                                                                                                                                                                      |
| `$nmap <IP> -v # OR -vv`    | Increase the verbosity level.                                                                                                                                                                            |

## Performance:

{% hint style="danger" %}
<mark style="color:red;">Optimized scans do accelerate the scanning process BUT can overlook important information.</mark>
{% endhint %}

{% embed url="https://nmap.org/book/man-performance.html" %}
Official Documentation.
{% endembed %}

### Optimizing Timeouts `--initial-rtt-timeout`

What's a timeout?\
It's the time between sending a packet and receiving a response.  (`Round-Trip-Time` - `RTT`)&#x20;

```bash
└──╼ $ sudo nmap 10.10.10.10/24 -F --initial-rtt-timeout 50ms --max-rtt-timeout 100ms
```

### Optimizing Max Retries `--max-retries`

By default, when nmap doesn't receive a response back, it re-sends a probe. That's called `--max-retries` option

```bash
└──╼ $ sudo nmap 10.10.10.10 -F --max-retries 0
```

### Optimizing sending rates (number of packets/sec)

You should know that:

* By default nmap does a good job of finding an appropriate speed to complete the scan.
* The fastest rate depends on the hardware.
* Scanning faster than a network can support may lead to a loss of accuracy and even longer waiting time then a slower rat

```bash
└──╼ $ sudo nmap 10.10.10.10 -F --mmin-rate = 100 --max-rate 400
```

### Optimizing Timings `-T <0-5>` option

These values (`0-5`) determine the <mark style="color:red;">**aggressiveness**</mark> of our scans.

* <mark style="color:green;">-T 0 / -T paranoid :</mark> For IDS evasion.
* <mark style="color:blue;">-T 1 / -T sneaky :</mark> For IDS evasion.
* <mark style="color:purple;">-T 2 / -T polite :</mark> Uses less bandwidth.
* <mark style="color:yellow;">-T 3 / -T :</mark> normal
* <mark style="color:orange;">-T 4 / -T aggressive :</mark> Assumes that you are on a fast and reliable network.
* <mark style="color:red;">**-T 5 / -T insane :**</mark> Assumes that you are on an extraordinarily fast network or are willing to sacrifice some accuracy for speed.

