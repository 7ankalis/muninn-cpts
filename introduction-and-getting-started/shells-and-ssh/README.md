---
description: >-
  This is a non-technical page about terms and tools we'll encounter all along
  the way.
---

# Shells & SSH

## SSH:

### Why SSH ?

Developed in 1995 by Tatu Ylönen, SSH, or Secure Shell, operating on its <mark style="color:green;">default port 22</mark>, is a cryptographic network protocol used for secure communication over an unsecured network. Initially, SSH was designed to provide secure remote access to Unix-like operating systems, allowing users to log into remote machines securely and execute commands as if they were sitting at the local console.

### SSH for Pentesters:

In general reverse shells aren't stable so an SSH connection would be more reliable . As we often run into cleartext credentials or an SSH private key, we can be use those to connect to the system in a more stable manner through SSH.

There  are two types of keys used in an SSH connection:&#x20;

* A Public key: This key is placed on the server we want to access. It's usually stored in a file called <mark style="color:green;">authorized\_keys</mark> in the <mark style="color:green;">.ssh</mark> directory within the user's home directory on the server.&#x20;
* A Private key: This key remains on our attacking machine and passed in the SSH command args.

The server checks if our **public key** is listed in the <mark style="color:green;">authorized\_keys</mark> file and if the corresponding <mark style="color:green;">private key</mark> is presented during the connection attempt. If both match, the server grants us access.\


## Shells:

### Shell types:

<table><thead><tr><th>Shell Type</th><th>Description</th><th data-hidden></th></tr></thead><tbody><tr><td><a href="./#reverse-shell">Reverse Shell</a></td><td>From Target To Attacker (Listener).</td><td></td></tr><tr><td><a href="./#bind-shell">Bind Shell</a> </td><td>From Attacker to Target (Listener: Binds to a port).</td><td></td></tr><tr><td><a href="./#web-shell">Web Shell</a></td><td>Runs operating system commands via the web browser, . Typically not interactive or semi-interactive and  can also be used to run single commands.</td><td></td></tr></tbody></table>

### Reverse shell :

<img src="../../.gitbook/assets/file.excalidraw (9).svg" alt="Reverse Shell illustration" class="gitbook-drawing">

{% hint style="info" %}
A Reverse Shell is handy when we want to get a quick, reliable connection to our compromised host. However, a Reverse Shell can be <mark style="color:blue;">very fragile</mark>. Once the reverse shell command is stopped, or if we lose our connection for any reason, we would have to use the initial exploit to execute the reverse shell command again to regain our access.
{% endhint %}

### Bind Shell :

<img src="../../.gitbook/assets/file.excalidraw (10).svg" alt="" class="gitbook-drawing">

{% hint style="info" %}
As we can see, we are directly dropped into a bash session and can interact with the target system directly. **Unlike a Reverse Shell**, if we drop our connection to a bind shell for any reason, we can connect back to it and get another connection immediately. However, if the bind shell command is stopped for any reason, or if the remote host is rebooted, we would still lose our access to the remote host and will have to exploit it again to gain access.
{% endhint %}

### Web Shell:

A Web Shell is typically a web script, i.e., `PHP` or `ASPX`, that accepts our command <mark style="color:blue;">through HTTP request parameters</mark> such as `GET` or `POST` request parameters, executes our command, and prints its output back on the web page.

{% hint style="info" %}
Each command we execute requests another URL to execute the commands.
{% endhint %}

<img src="../../.gitbook/assets/file.excalidraw (11).svg" alt="" class="gitbook-drawing">
