---
description: >-
  This page contains non technical information about the concept of shells and
  SSH.
---

# Shells & SSH

## SSH:

### Why SSH ?

SSH, or Secure Shell, is a cryptographic network protocol developed in 1995 by Tatu Ylönen. Operating on its default port, 22, SSH enables secure communication over unsecured networks.

Initially designed for secure remote access to Unix-like operating systems, SSH allows users to log into remote machines and execute commands as if they were working on a local console, providing a secure and efficient way to manage remote systems.

### SSH for Pentesters:

Reverse shells are often unstable, making SSH connections a more reliable alternative for maintaining access. When pentesters encounter cleartext credentials or an SSH private key, they can use these to establish a stable connection to the target system via SSH.

**Types of SSH Keys**

1. **Public Key**:
   * Stored on the target server.
   * Typically saved in the `authorized_keys` file located in the `.ssh` directory within the user's home directory.
2. **Private Key**:
   * Remains on the attacker’s machine.
   * Passed as an argument in the SSH command during connection.

When connecting via SSH, the server checks whether the public key matches one listed in the `authorized_keys` file. If the corresponding private key is presented and verified, the server grants access, ensuring a secure and stable connection.\


## Shells:

### Shell types:

<table><thead><tr><th>Shell Type</th><th>Description</th><th data-hidden></th></tr></thead><tbody><tr><td><a href="./#reverse-shell">Reverse Shell</a></td><td>From Target To Attacker (Listener).</td><td></td></tr><tr><td><a href="./#bind-shell">Bind Shell</a> </td><td>From Attacker to Target (Listener: Binds to a port).</td><td></td></tr><tr><td><a href="./#web-shell">Web Shell</a></td><td>Runs operating system commands via the web browser, . Typically not interactive or semi-interactive and  can also be used to run single commands.</td><td></td></tr></tbody></table>

### Reverse shell :

<img src="../../.gitbook/assets/file.excalidraw (9).svg" alt="Reverse Shell illustration with a php file." class="gitbook-drawing">

{% hint style="info" %}
A Reverse Shell is handy when we want to get a quick, reliable connection to our compromised host. However, a Reverse Shell can be <mark style="color:blue;">very fragile</mark>. Once the reverse shell command is stopped, or if we lose our connection for any reason, we would have to use the initial exploit to execute the reverse shell command again to regain our access and this is where SSH come in handy, since it stable most of the times, responsive and interactive.
{% endhint %}

### Bind Shell :

<img src="../../.gitbook/assets/file.excalidraw (10).svg" alt="" class="gitbook-drawing">

{% hint style="info" %}
As we can see, we are directly dropped into a bash session and can interact with the target system directly. **Unlike a Reverse Shell**, if we drop our connection to a bind shell for any reason, we can connect back to it and get another connection immediately. However, if the bind shell command is stopped for any reason, or if the remote host is rebooted, we would still lose our access to the remote host and will have to exploit it again to gain access.
{% endhint %}

### Web Shell:

A Web Shell is typically a web script, i.e., `PHP` or `ASPX`, that accepts our command <mark style="color:blue;">through HTTP request parameters</mark> such as `GET` or `POST` request parameters, executes our command, and prints its output back on the web page. So our commands simply get executed on the server and the response is sent back to be rendered on the page. So if we send a Reverse Shell command to it and successfully gets executed, expect a non responsive web page and a hit back on the listener we set up on our machine beforehand. We can look at it as if the process that handles our request is on our hook managing the connection IT initialized.&#x20;

{% hint style="info" %}
Each command we execute requests another URL to execute the commands.
{% endhint %}

<img src="../../.gitbook/assets/file.excalidraw (11).svg" alt="" class="gitbook-drawing">
