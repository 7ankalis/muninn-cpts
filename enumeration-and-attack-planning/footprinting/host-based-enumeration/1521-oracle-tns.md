---
description: Transport Network Substrate
---

# (1521)Oracle TNS

{% hint style="info" %}
<mark style="color:blue;">This document will be published upon completion of the module.</mark>
{% endhint %}

## About

The Oracle Transparent Network Substrate (TNS) is a communication protocol **designed to facilitate interactions between Oracle databases and applications over networks**. Initially introduced as part of the Oracle Net Services suite, it supports various networking protocols, such as IPX/SPX and TCP/IP, making it a preferred choice for managing large, complex databases in industries like healthcare, finance, and retail. Its built-in encryption ensures secure data transmission, making it ideal for enterprise environments with high data security needs.

Over time, TNS has evolved to support newer technologies like IPv6 and SSL/TLS encryption, further enhancing its functionality. These updates make TNS suitable for tasks such as:

* Name resolution
* Connection management
* Load balancing
* Security enhancement

TNS adds an extra layer of encryption between client-server communication over the TCP/IP protocol, helping secure databases from unauthorized access and network traffic attacks. Additionally, it offers advanced tools for database administrators and developers, including performance monitoring, error reporting, workload management, and fault tolerance through database services.

### Default Configuration

{% hint style="warning" %}
You should know that the listening port can be changed during installation.&#x20;
{% endhint %}

By default and depending on the version, the TNS listener includes certain security features like a default password, allows connection from the only authorized hosts, perform basic authentication, uses Oracle Net Services to encrypt communication between the client and server.

The configuration files for Oracle TNS are called `tnsnames.ora` and `listener.ora` .

### tnsnames.ora

Each database or service has a unique entry in the [tnsnames.ora](https://docs.oracle.com/cd/E11882\_01/network.112/e10835/tnsnames.htm#NETRF007) file, containing the necessary information for clients to connect to the service. The entry consists of a name for the service, the network location of the service, and the database or service name that clients should use when connecting to the service.&#x20;

* **Client-side configuration**.
* Defines the network service names and connection details used by clients to connect to Oracle databases.
* Maps human-readable service names to network addresses (e.g., database name, protocol, host, port).
* Used by clients to locate and connect to the appropriate Oracle database instance.

For example, a simple `tnsnames.ora` file might look like this:

```txt
ORCL =  <- Service name
  (DESCRIPTION =
    (ADDRESS_LIST =
      (ADDRESS = (PROTOCOL = TCP)(HOST = 10.129.11.102)(PORT = 1521)) <- IP+Port to connect to
    )
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = orcl) <- clients should use the name orcl when connecting to the service.
    )
  )
```

### listener.ora

The `listener.ora` file is a server-side configuration file that specifies the properties and parameters of the listener process, which handles incoming client requests and routes them to the correct Oracle database instance.

* **Server-side configuration**.
* Specifies the settings for the Oracle listener, a process that receives incoming client connection requests.
* Defines the listener's protocol addresses (e.g., TCP/IP, port) and the Oracle database instances it services.
* Controls how client requests are routed to the appropriate database instance.



```txt
SID_LIST_LISTENER =
  (SID_LIST =
    (SID_DESC =
      (SID_NAME = PDB1)
      (ORACLE_HOME = C:\oracle\product\19.0.0\dbhome_1)
      (GLOBAL_DBNAME = PDB1)
      (SID_DIRECTORY_LIST =
        (SID_DIRECTORY =
          (DIRECTORY_TYPE = TNS_ADMIN)
          (DIRECTORY = C:\oracle\product\19.0.0\dbhome_1\network\admin)
        )
      )
    )
  )

LISTENER =
  (DESCRIPTION_LIST =
    (DESCRIPTION =
      (ADDRESS = (PROTOCOL = TCP)(HOST = orcl.inlanefreight.htb)(PORT = 1521))
      (ADDRESS = (PROTOCOL = IPC)(KEY = EXTPROC1521))
    )
  )

ADR_BASE_LISTENER = C:\oracle
```

> In short, the client-side Oracle Net Services software uses the `tnsnames.ora` file to resolve service names to network addresses, while the listener process uses the `listener.ora` file to determine the services it should listen to and the behavior of the listener.

## Footprinting

Before installation, we should install dependencies using this bash script:

```bash
#!/bin/bash

sudo apt-get install libaio1 python3-dev alien -y
git clone https://github.com/quentinhardy/odat.git
cd odat/
git submodule init
git submodule update
wget https://download.oracle.com/otn_software/linux/instantclient/2112000/instantclient-basic-linux.x64-21.12.0.0.0dbru.zip
unzip instantclient-basic-linux.x64-21.12.0.0.0dbru.zip
wget https://download.oracle.com/otn_software/linux/instantclient/2112000/instantclient-sqlplus-linux.x64-21.12.0.0.0dbru.zip
unzip instantclient-sqlplus-linux.x64-21.12.0.0.0dbru.zip
export LD_LIBRARY_PATH=instantclient_21_12:$LD_LIBRARY_PATH
export PATH=$LD_LIBRARY_PATH:$PATH
pip3 install cx_Oracle
sudo apt-get install python3-scapy -y
sudo pip3 install colorlog termcolor passlib python-libnmap
sudo apt-get install build-essential libgmp-dev -y
pip3 install pycryptodome
```

```shell-session
$ ./odat.py -h
```

### Nmap

```shell-session
$ sudo nmap -p1521 -sV <TARGET> --open
```

To connect to a database in ODBMS we will need a System Identifier (SID) which is the unique name to identify the database. So intuitively, not providing the correct SID will lead the connection to fail. So we need an SID which we will obtain using a bruteforcing tool (nmap, odat, hydra ..etc)

{% hint style="info" %}
Although calling Nmap a bruteforcing isn't technically correct, but we will using that feature.
{% endhint %}

### SID Bruteforcing

#### Nmap

```shell-session
$ sudo nmap -p1521 -sV <TARGET> --open --script oracle-sid-brute
```

#### odat.py

```shell-session
$ ./odat.py all -s <TARGET>

[+] Checking if target 10.129.204.235:1521 is well configured for a connection...
[+] According to a test, the TNS listener 10.129.204.235:1521 is well configured. Continue...

...SNIP...

[!] Notice: 'mdsys' account is locked, so skipping this username for password           #####################| ETA:  00:01:16 
[!] Notice: 'oracle_ocm' account is locked, so skipping this username for password       #####################| ETA:  00:01:05 
[!] Notice: 'outln' account is locked, so skipping this username for password           #####################| ETA:  00:00:59
[+] Valid credentials found: scott/tiger. Continue...

...SNIP...
```

{% hint style="success" %}
Upon finding correct credentials, we're able to connect to the database.
{% endhint %}

### Interacting with the server

```shell-session
$ sqlplus username/password@TARGET/SID
sqlplus: error while loading shared libraries: libsqlplus.so: cannot open shared object file: No such file or directory 

$ sudo sh -c "echo /usr/lib/oracle/12.2/client64/lib > /etc/ld.so.conf.d/oracle-instantclient.conf";sudo ldconfig
```

With a bit of  [SQLplus commands](https://docs.oracle.com/cd/E11882\_01/server.112/e41085/sqlqraa001.htm#SQLQR985) we can move around and footprint the databases.

```sql
SQL> select table_name from all_tables;
SQL> select * from <tablename>;
```

We can try to connect as an administrator from the obtained usernames above:

```shell-session
$ sqlplus scott/tiger@10.129.204.235/XE as sysdba 
```

If this works we can enumerate further:

```sql
SQL> select * from user_role_privs;
```

#### Password hashes retrieval

```sql
SQL> select name, password from sys.user$;
```

{% hint style="info" %}
Another approach consists of uploading a web shell only if the server is running a webserver and we know its [root directory](../../../introduction-and-getting-started/shells-and-ssh/setting-up-shells.md#id-3.-web-shells). Here's how it goes:

First, we try to inject a non malicious file to see if it goes through IDS/IPS systems or Antiviruses and then test if the injection was successful.

```shell-session
$ ./odat.py utlfile -s <TARGET> -d XE -U username -P password --sysdba --putFile C:\\inetpub\\wwwroot testing.txt ./testing.txt
$ curl -X GET http://10.129.204.235/testing.txt
```
{% endhint %}

