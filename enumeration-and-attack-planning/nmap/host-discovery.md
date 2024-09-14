# Host Discovery

There are several scanning types nmap can detect whether a host is up or not. By default, nmap uses ARP pinging method, which isn't the "best" for our case.

We'll use the ICMP echo requests which is the most effective host discovery method. So, we must disable ARP pinging, and enable ICMP echo requests.

```bash
└──╼ $nmap 10.10.10.10 --disable-arp-ping -PE -ns 
    --diable-arp-ping : self-explanatory.
    -PE : enable ICMP echo requests.
    -n : Do not perform any DNS lookup (speeds up the scan).
    -s : Do not perform port scanning.
```

{% hint style="info" %}
It must be mentioned that nmap offers a LOT more than just echo requests and ARP pinging. For more host discovery techniques, visit the [official nmap documentation.](https://nmap.org/book/host-discovery-techniques.html)
{% endhint %}
