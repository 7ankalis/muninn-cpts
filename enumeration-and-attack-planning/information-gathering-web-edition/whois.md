# WHOIS

In short, it's the phonebook for the internet that lets us look up who owns/responsible for various online assets.

## Why using WHOIS ?

1. Identifying Key Personnel: names, email addresses, and phone numbers of individuals responsible for managing the domain.
2. Discovering Network infrastructure: name servers and IP addresses provide clues about the target's network infrastructure.
3. Historical data Analysis: Accessing historical WHOIS records through services like [WhoisFreaks](https://whoisfreaks.com/) can reveal changes in ownership, contact information, or technical details over time. This can be useful for tracking the evolution of the target's digital presence.

In conclusion, using WHOIS can help us investigate emails that redirects us to domains that claim to be valid when they're not, investigate a C2 server in a malware analysis scenario or even in generating a threat intelligence report about a threat actor group to detect and prevent future attacks.

## Utilizing WHOIS

> While the WHOIS record provides contact information for domain-related issues, it might not be directly helpful in identifying individual employees or specific vulnerabilities. This highlights the need to combine WHOIS data with other reconnaissance techniques to understand the target's digital footprint comprehensively.

```shell-session
$ whois <domainname>
```
