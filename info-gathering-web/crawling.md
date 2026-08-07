# Crawling

## How Web Crawlers Work

Stats with seed URL, parses its content, extracts all links, and adds them to a queue to explore

**Breadth-First Crawling** explores widely before going deep. Explores all links on homepage before going any deeper

![Flowchart showing a Seed URL leading to Page 1, which branches to Page 2 and Page 3. Page 2 connects to Page 4 and Page 5, while Page 3 connects to Page 6 and Page 7.](https://academy.hackthebox.com/storage/modules/144/ig_crawling_1.png)

**Depth-First Crawling** prioritizes depth over breaths. Follows a path of links as far as possible before backtracking and exploring other paths

## Extracting Valuable Information

**Links (Internal and External), Comments, Metadata, Sensitive Files (backups, configs, logs, creds, keys)**

One piece of info from a comment can be combined with other findings to piece together a possible vulnerability

For instance, a list of extracted links might initially appear mundane. But upon closer examination, you notice a pattern: several URLs point to a directory named `/files/`. This triggers your curiosity, and you decide to manually visit the directory. To your surprise, you find that directory browsing is enabled

## Robots.txt

Access and off-limits list for bots

### Understand the Structure

1. `User-agent`: This line specifies which crawler or bot the following rules apply to. A wildcard (`*`) indicates that the rules apply to all bots. Specific user agents can also be targeted, such as "Googlebot" (Google's crawler) or "Bingbot" (Microsoft's crawler).
2. `Directives`: These lines provide specific instructions to the identified user-agent.

Common directives include:

| Directive     | Description                                                                                                        | Example                                                      |
| ------------- | ------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------ |
| `Disallow`    | Specifies paths or patterns that the bot should not crawl.                                                         | `Disallow: /admin/` (disallow access to the admin directory) |
| `Allow`       | Explicitly permits the bot to crawl specific paths or patterns, even if they fall under a broader `Disallow` rule. | `Allow: /public/` (allow access to the public directory)     |
| `Crawl-delay` | Sets a delay (in seconds) between successive requests from the bot to avoid overloading the server.                | `Crawl-delay: 10` (10-second delay between requests)         |
| `Sitemap`     | Provides the URL to an XML sitemap for more efficient crawling.                                                    | `Sitemap: https://www.example.com/sitemap.xml`               |

### robots.txt in Web Recon

* `Uncovering Hidden Directories`: Disallowed paths in robots.txt often point to directories or files the website owner intentionally wants to keep out of reach from search engine crawlers. These hidden areas might house sensitive information, backup files, administrative panels, or other resources that could interest an attacker.
* `Mapping Website Structure`: By analyzing the allowed and disallowed paths, security professionals can create a rudimentary map of the website's structure. This can reveal sections that are not linked from the main navigation, potentially leading to undiscovered pages or functionalities.
* `Detecting Crawler Traps`: Some websites intentionally include "honeypot" directories in robots.txt to lure malicious bots. Identifying such traps can provide insights into the target's security awareness and defensive measures.

## .Well-Known URIs

The `.well-known` standard, defined in [RFC 8615](https://datatracker.ietf.org/doc/html/rfc8615), serves as a standardized directory within a website's root domain. This designated location, typically accessible via the `/.well-known/` path on a web server, **centralizes a website's critical metadata**, including configuration files and information related to its services, protocols, and security mechanisms

Enables clients to automatically locate and retrieve specific configuration files by constructing the appropriate URL. For instance, to access a website's security policy, a client would request `https://example.com/.well-known/security.txt`

The `Internet Assigned Numbers Authority` (`IANA`) maintains a [registry](https://www.iana.org/assignments/well-known-uris/well-known-uris.xhtml) of `.well-known` URIs. Here are some:

| URI Suffix                     | Description                                                                                           | Status      | Reference                                                                               |
| ------------------------------ | ----------------------------------------------------------------------------------------------------- | ----------- | --------------------------------------------------------------------------------------- |
| `security.txt`                 | Contains contact information for security researchers to report vulnerabilities.                      | Permanent   | RFC 9116                                                                                |
| `/.well-known/change-password` | Provides a standard URL for directing users to a password change page.                                | Provisional | https://w3c.github.io/webappsec-change-password-url/#the-change-password-well-known-uri |
| `openid-configuration`         | Defines configuration details for OpenID Connect, an identity layer on top of the OAuth 2.0 protocol. | Permanent   | http://openid.net/specs/openid-connect-discovery-1\_0.html                              |
| `assetlinks.json`              | Used for verifying ownership of digital assets (e.g., apps) associated with a domain.                 | Permanent   | https://github.com/google/digitalassetlinks/blob/master/well-known/specification.md     |
| `mta-sts.txt`                  | Specifies the policy for SMTP MTA Strict Transport Security (MTA-STS) to enhance email security.      | Permanent   | RFC 8461                                                                                |

### Well-known Web Recon

`.well-known` URIs can be invaluable for discovering endpoints and configuration details that can be further tested during a penetration test. One particularly useful URI is `openid-configuration`

We can use the `https://example.com/.well-known/openid-configuration` endpoint to get a lot of info:

* `Endpoint Discovery`:
  * `Authorization Endpoint`: Identifying the URL for user authorization requests.
  * `Token Endpoint`: Finding the URL where tokens are issued.
  * `Userinfo Endpoint`: Locating the endpoint that provides user information.
* `JWKS URI`: The `jwks_uri` reveals the `JSON Web Key Set` (`JWKS`), detailing the cryptographic keys used by the server.
* `Supported Scopes and Response Types`: Understanding which scopes and response types are supported helps in mapping out the functionality and limitations of the OpenID Connect implementation.
* `Algorithm Details`: Information about supported signing algorithms can be crucial for understanding the security measures in place.

## Web Crawl Tools

* `Burp Suite Spider`: Burp Suite, a widely used web application testing platform, includes a powerful active crawler called Spider. Spider excels at mapping out web applications, identifying hidden content, and uncovering potential vulnerabilities.
* `OWASP ZAP (Zed Attack Proxy)`: ZAP is a free, open-source web application security scanner. It can be used in automated and manual modes and includes a spider component to crawl web applications and identify potential vulnerabilities.
* `Scrapy (Python Framework)`: Scrapy is a versatile and scalable Python framework for building custom web crawlers. It provides rich features for extracting structured data from websites, handling complex crawling scenarios, and automating data processing. Its flexibility makes it ideal for tailored reconnaissance tasks.
* `Apache Nutch (Scalable Crawler)`: Nutch is a highly extensible and scalable open-source web crawler written in Java. It's designed to handle massive crawls across the entire web or focus on specific domains. While it requires more technical expertise to set up and configure, its power and flexibility make it a valuable asset for large-scale reconnaissance projects.

### Scrapy

Install:

```bash
pip3 install scrapy
```

First, run this to download the custom scrapy spider ReconSpider

```bash
wget -O ReconSpider.zip https://academy.hackthebox.com/storage/modules/144/ReconSpider.v1.2.zip
unzip ReconSpider.zip 
python3 ReconSpider.py <domain>
```

After running ReconSpider, it saves data in results.json

```json
{
    "emails": [
        "lily.floid@inlanefreight.com",
        "cvs@inlanefreight.com",
        ...
    ],
    "links": [
        "https://www.themeansar.com",
        "https://www.inlanefreight.com/index.php/offices/",
        ...
    ],
    "external_files": [
        "https://www.inlanefreight.com/wp-content/uploads/2020/09/goals.pdf",
        ...
    ],
    "js_files": [
        "https://www.inlanefreight.com/wp-includes/js/jquery/jquery-migrate.min.js?ver=3.3.2",
        ...
    ],
    "form_fields": [],
    "images": [
        "https://www.inlanefreight.com/wp-content/uploads/2021/03/AboutUs_01-1024x810.png",
        ...
    ],
    "videos": [],
    "audio": [],
    "comments": [
        "<!-- #masthead -->",
        ...
    ]
}
```

