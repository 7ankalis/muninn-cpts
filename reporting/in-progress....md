# In-Progress...

{% hint style="info" %}
<mark style="color:blue;">This document will be published upon completion of the module</mark>
{% endhint %}

Most host-based anti-virus software nowadays relies mainly on `Signature-based Detection` to identify aspects of malicious code present in a software sample. These signatures are placed inside the Antivirus Engine, where they are subsequently used to scan storage space and running processes for any matches. When a piece of unknown software lands on a partition and is matched by the Antivirus software, most Anti-viruses quarantine the malicious program and kill the running process.

How do we circumvent all this heat? We play along with it. The examples shown in the `Encoders` section show that simply encoding payloads using different encoding schemes with multiple iterations is not enough for all AV products. Moreover, merely establishing a channel of communication between the attacker and the victim can raise some alarms with the current capabilities of IDS/IPS products out there.

However, with the MSF6 release, msfconsole can tunnel AES-encrypted communication from any Meterpreter shell back to the attacker host, successfully encrypting the traffic as the payload is sent to the victim host. This mostly takes care of the network-based IDS/IPS. In some rare cases, we might be met with very strict traffic rulesets that flag our connection based on the sender's IP address. The only way to circumvent this is to find the services being let through. An excellent example of this would be the Equifax hack of 2017, where malicious hackers have abused the Apache Struts vulnerability to access a network of critical data servers. DNS exfiltration techniques were used to slowly siphon data out of the network and into the hackers' domain without being noticed for months. To learn more about this attack, visit the links below:

{% embed url="https://www.zdnet.com/article/us-government-releases-post-mortem-report-on-equifax-hack/" %}

{% embed url="https://www.darkreading.com/cyber-risk/tips-to-protect-the-dns-from-data-exfiltration" %}

