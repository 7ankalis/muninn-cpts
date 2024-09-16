# SNMP(161 UDP)

## About

> SNMP was created to monitor network devices. In addition, this protocol can also be used to handle configuration tasks and change settings remotely. SNMP-enabled hardware includes routers, switches, servers, IoT devices, and many other devices that can also be queried and controlled using this standard protocol. Thus, it is a protocol for monitoring and managing network devices.

<img src="../../../.gitbook/assets/file.excalidraw (14).svg" alt="" class="gitbook-drawing">

It is important too note the the SNMP traps only happen if the device is configured to do so, and with no requests from the client.

* **SNMPv1**: Basic version with no security (no authentication or encryption), suitable for small networks.
* **SNMPv2c**: Added features but still insecure due to lack of encryption and use of community strings in plain text.
* **SNMPv3**: Improved security with authentication and encryption, though it comes with increased configuration complexity.

### Addressing Mechanism

<img src="../../../.gitbook/assets/file.excalidraw (15).svg" alt="" class="gitbook-drawing">

## Default Configuration

```shell-session
$ cat /etc/snmp/snmpd.conf | grep -v "#" | sed -r '/^\s*$/d'

sysLocation    Sitting on the Dock of the Bay
sysContact     Me <me@example.org>
sysServices    72
master  agentx
agentaddress  127.0.0.1,[::1]
view   systemonly  included   .1.3.6.1.2.1.1
view   systemonly  included   .1.3.6.1.2.1.25.1
rocommunity  public default -V systemonly
rocommunity6 public default -V systemonly
rouser authPrivUser authpriv -V systemonly
```

### Dangerous Settings

| <mark style="color:red;">**rwuser noauth**</mark>                                    | Provides access to the full OID tree without authentication.                          |
| ------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------- |
| <mark style="color:red;">**rwcommunity \<community string> \<IPv4 address>**</mark>  | Provides access to the full OID tree regardless of where the requests were sent from. |
| <mark style="color:red;">**rwcommunity6 \<community string> \<IPv6 address>**</mark> | Same access as with `rwcommunity` with the difference of using IPv6.                  |

## Footprinting

### snmpwalk

The command below will return a list of SNMP OIDs (Object Identifiers) and their values, which represent the various data points the device is exposing.

{% hint style="info" %}
<mark style="color:blue;">We will need a community string and an SNMP version that does not support authentication!</mark>
{% endhint %}

```shell-session
$ snmpwalk -v2c -c public <TARGET IP>
    -v2c : Specifies the SNMP server version (2c in this situation)
    -c : Specifies the community string.
```

If we do not have a community string:

### onesixtyone

This tool brute forces the community strings using wordlists of our choice.

```shell-session
$ sudo apt install onesixtyone
$ onesixtyone -c <PATH TO WORDLIST> <TARGET IP>
```

{% hint style="info" %}
We can either create our custom wordlist or use one from SecLists \
(/SecLists/Discovery/SNMP/snmp.txt)
{% endhint %}

### braa

As said in the documentation of this tool :

> ```
>   Braa is a mass snmp scanner. The intended usage of such a tool is of course 
> making SNMP queries - but unlike snmpget or snmpwalk from net-snmp, it is able
> to query dozens or hundreds of hosts simultaneously, and in a single process.
> Thus, it consumes very few system resources and does the scanning VERY fast.
>   
>   Braa implements its OWN snmp stack, so it does NOT need any SNMP libraries
> like net-snmp. The implementation is very dirty, supports only several data
> types, and in any case cannot be stated 'standard-conforming'! It was designed
> to be fast, and it is fast. For this reason (well, and also because of my
> laziness ;), there is no ASN.1 parser in braa - you HAVE to know the numerical
> values of OID's (for instance .1.3.6.1.2.1.1.5.0 instead of system.sysName.0).
> ```

So in short, it's a tool to to brute-force the individual OIDs and enumerate the information behind them.

```shell-session
$ sudo apt install braa
$ braa <community string>@<IP>:.1.3.6.*   # Syntax
$ braa public@10.129.14.128:.1.3.6.*
```
