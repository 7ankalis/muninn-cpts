# SMTP(25/465(enc)/587(newer))

## About

* Often used with IMAP/POP3 which we will discuss later and provide the full image of the mail transferring action.
* SMTP also prevents spam with its authentication mechanisms through supported the extension ESMTP and SMTP-Auth.
* Not encrypted mainly, but can use SSL/TLS encryption on port 465 by ESTMP which uses TLS after the EHLO command.

<img src="../../../.gitbook/assets/file.excalidraw (1) (1).svg" alt="General illustration on how SMTP is used." class="gitbook-drawing">

Without diving into IMAP/POP3 to get the full image of how the mailing process works, here's how smtp is used in general:

<img src="../../../.gitbook/assets/file.excalidraw (2) (1).svg" alt="" class="gitbook-drawing">

* Uses DKIM+SPF to prevent spam and the SMTP relays attacks.

## Config

## Enumeration

| `AUTH PLAIN`                           | AUTH is a service extension used to authenticate the client.                                      |
| -------------------------------------- | ------------------------------------------------------------------------------------------------- |
| `HELO`                                 | The client logs in with its computer name and thus starts the session.                            |
| `MAIL FROM`                            | The client names the email sender.                                                                |
| `RCPT TO`                              | The client names the email recipient.                                                             |
| `DATA`                                 | The client initiates the transmission of the email.                                               |
| `RSET`                                 | The client aborts the initiated transmission but keeps the connection between client and server.  |
| <mark style="color:red;">`VRFY`</mark> | <mark style="color:red;">The client checks if a mailbox is available for message transfer.</mark> |
| `EXPN`                                 | The client also checks if a mailbox is available for messaging with this command.                 |
| `NOOP`                                 | The client requests a response from the server to prevent disconnection due to time-out.          |
| `QUIT`                                 | The client terminates the session.                                                                |

### Interaction

#### Telnet + HELO/EHLO

```bash
$ telnet 10.10.10.10 25
Trying 10.10.10.10...
Connected to 10.10.10.10.
Escape character is '^]'.
220 ESMTP Server 

# HELO is the original SMTP command used to greet and initialize the connection.
# HELO simply indicates that the client wishes to start an email transaction and provides the client's domain name or IP address.

HELO mail1.inlanefreight.htb

250 mail1.inlanefreight.htb

# The EHLO (Extended HELO) command is an extended version of the HELO command.
EHLO mail1
# Listing the supported ESMTP extensions.

250-mail1.inlanefreight.htb
250-PIPELINING
250-SIZE 10240000
250-ETRN
250-ENHANCEDSTATUSCODES
250-8BITMIME
250-DSN
250-SMTPUTF8
250 CHUNKING
```

#### <mark style="color:red;">Telnet + VRFY</mark>

The <mark style="color:red;">VRFY</mark> command, like any other command uses pre-configured commands. Therefore the false positives are so common especially using <mark style="color:red;">VRFY.</mark> Here's how:

```bash
$ telnet 10.10.10.10 25

Trying 10.10.10.10...
Connected to 10.10.10.10.
Escape character is '^]'.
220 ESMTP Server 

VRFY root

252 2.0.0 root


VRFY cry0l1t3

252 2.0.0 cry0l1t3


VRFY testuser

252 2.0.0 testuser


>>>>> VRFY aaaaaaaaaaaaaaaaaaaaaaaaaaaa

>>>>> 252 2.0.0 aaaaaaaaaaaaaaaaaaaaaaaaaaaa
```

### Nmap

```bash
$ sudo nmap 10.10.10.10 -sC -sV -p25
$ sudo nmap 10.10.10.10 -p25 --script smtp-open-relay -v
```
