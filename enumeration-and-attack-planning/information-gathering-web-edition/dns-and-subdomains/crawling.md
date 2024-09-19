# Crawling

## About

In short, it's literally crawling the webpage (called seed page)  for any sub-pages and links to extract valuable information such us comments, metadata, sensitive files and exposing potential vulnerabilities to exploit.

## robots.txt

Technically, robots.txt is a text file placed in the root directory of a website that tells web crawlers which directories they can or cannot crawl through directives that adhere to the Robots Exclusion Standard.

So for our reconnaissance robots.txt can provide us with hidden directories (maybe login portals?), help us through mapping the website structure.

```
User-agent: *
Disallow: /admin/ : Disallowed entry.
Disallow: /private/ : Disallowed entry.
Allow: /public/

User-agent: Googlebot
Crawl-delay: 10 : Delay in seconds between request.

Sitemap: https://www.example.com/sitemap.xml # For better crawling and indexing.
```

## Well-known URLs

A well-known URL is a standardized URL path used to access specific information on a website, often for system discovery or configuration purposes. These URLs typically start with `/.well-known/` followed by the specific service name. Examples include:

* `/.well-known/security.txt` — contains security-related information for reporting vulnerabilities.
* `/.well-known/openid-configuration` — provides OpenID Connect configuration data.

They help streamline and standardize processes like security, authentication, or metadata sharing across websites.

Here's what the **openid-configuration** directory could expose:

> The `openid-configuration` URI is part of the OpenID Connect Discovery protocol, an identity layer built on top of the OAuth 2.0 protocol. When a client application wants to use OpenID Connect for authentication, it can retrieve the OpenID Connect Provider's configuration by accessing the `https://example.com/.well-known/openid-configuration` endpoint. This endpoint returns a JSON document containing metadata about the provider's endpoints, supported authentication methods, token issuance, and more:

```json
{
  "issuer": "https://example.com",
  "authorization_endpoint": "https://example.com/oauth2/authorize",
  "token_endpoint": "https://example.com/oauth2/token",
  "userinfo_endpoint": "https://example.com/oauth2/userinfo",
  "jwks_uri": "https://example.com/oauth2/jwks",
  "response_types_supported": ["code", "token", "id_token"],
  "subject_types_supported": ["public"],
  "id_token_signing_alg_values_supported": ["RS256"],
  "scopes_supported": ["openid", "profile", "email"]
}
```

>
>
> The information obtained from the `openid-configuration` endpoint provides multiple exploration opportunities:
>
> 1. `Endpoint Discovery`:
>    * `Authorization Endpoint`: Identifying the URL for user authorization requests.
>    * `Token Endpoint`: Finding the URL where tokens are issued.
>    * `Userinfo Endpoint`: Locating the endpoint that provides user information.
> 2. `JWKS URI`: The `jwks_uri` reveals the `JSON Web Key Set` (`JWKS`), detailing the cryptographic keys used by the server.
> 3. `Supported Scopes and Response Types`: Understanding which scopes and response types are supported helps in mapping out the functionality and limitations of the OpenID Connect implementation.
> 4. `Algorithm Details`: Information about supported signing algorithms can be crucial for understanding the security measures in place.

## Tools

```shell-session
$ pip3 install scrapy
$ wget -O ReconSpider.zip https://academy.hackthebox.com/storage/modules/144/ReconSpider.v1.2.zip
$ python3 ReconSpider.py <TARGET>
```

This will result in a JSON file containing the outcome of the web spidering.

