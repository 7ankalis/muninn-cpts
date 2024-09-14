# NSE

## Update the NSE scripts database

```bash
└──╼ $sudo nmap --script-updatedb
```

## Search for available scripts with a certain word

```bash
└──╼ $sudo find / -name dns* 2>/dev/null | grep scripts
```

## Trace NSE script sent

```bash
└──╼ $sudo nmap --script-trace
```

## Useful NSE scripts

### SMTP + Banner Grabbing

```bash
└──╼ $sudo nmap 10.10.10.10 -p 25 --script banner,smtp-commands
```

### Aggressive scan

```bash
└──╼ $sudo nmap 10.10.10.10 -p 80 -A
```

It does all the following :&#x20;

* Service Detection `-sV`
* OS Detection `-O`
* Trace the probes' routes `--traceroute`
* Default NSE scripts `-sC`

### Choose scripts by category

```bash
└──╼ $sudo nmap 10.10.10.10 -p 80 -sV --script vuln 
```

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption><p>TOP 10 useful scripts according to Station X</p></figcaption></figure>
