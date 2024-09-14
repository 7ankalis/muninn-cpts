---
description: Uses TCP, maybe that's why it informs us when the transfer wasn't successful?
---

# FTP(21)

## About:

<img src="../../../.gitbook/assets/file.excalidraw (3).svg" alt="Basic FTP connection graph." class="gitbook-drawing">

### Active VS Passive:

<img src="../../../.gitbook/assets/file.excalidraw (5).svg" alt="" class="gitbook-drawing">

{% hint style="info" %}
TFTP on the other hand, is a more basic implementation of FTP:

* It uses UDP.
* Needs no user authentication.
* No directory listing functionality

These characteristics are enough to assume that this protocol should be used only on strongly secured networks.
{% endhint %}

## Config:

For Linux, it is common to use the vsFTPd server to implement the FTP protocol.

```bash
$ sudo apt install vsftpd
```

* _**/etc/vstpd.conf**_
* _**/etc/ftpusers : Blocked users**_

### Useful commands:

* ls
* debug/trace
* get/put (if it's enabled)
* ls -R (if it's enabled)&#x20;

```bash
$ wget -m --no-passive ftp://username:password@targetIP

## Outputs a directory with the target IP name containing the downloaded content.
```

## Enumeration

### Nmap

```bash
$ sudo nmap  -p21 -sC --script=ftp* 10.10.10.10
```

### Interaction and banner grabbing:

<img src="../../../.gitbook/assets/file.excalidraw (7).svg" alt="" class="gitbook-drawing">

Upon connecting to the server, we'll get a response depending on how it was configured, we'll try to grab and check that banner to get more information about the server.

```bash
$nc 10.10.10.10 21
$telnet 10.10.10.10 21
```

If the server runs with TLS/SSL encryption:

```bash
$ openssl s_client -connect 10.10.10.10:21 -starttls ftp
```
