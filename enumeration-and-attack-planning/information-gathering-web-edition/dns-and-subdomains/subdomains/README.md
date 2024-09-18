# Subdomains

It is important for us as Pentesters to find and explore any hidden parts of the target's domain. That being said, subdomains represent a source of valuable information since they can hold hidden login portals, legacy apps, developement and staging enviroments..etc So there's a high chance we find a foothold through them.

We can find these subdomains with two approaches:

1. Active Enumeration: Brute forcing using wordlists of potential subdomain names against the target system.
2. Passive Enumeration: Look into Certificate Transparency logs, SSL/TLS certificates ..etc
