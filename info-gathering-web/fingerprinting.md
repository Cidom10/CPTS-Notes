# Fingerprinting

Fingerprinting focuses on extracting technical details about the technologies powering a website or web application

Fingerprinting serves as a cornerstone of web reconnaissance for several reasons:

* `Targeted Attacks`: By knowing the specific technologies in use, attackers can focus their efforts on exploits and vulnerabilities that are known to affect those systems. This significantly increases the chances of a successful compromise
* `Identifying Misconfigurations`: Fingerprinting can expose misconfigured or outdated software, default settings, or other weaknesses that might not be apparent through other reconnaissance methods
* `Prioritising Targets`: When faced with multiple potential targets, fingerprinting helps prioritise efforts by identifying systems more likely to be vulnerable or hold valuable information
* `Building a Comprehensive Profile`: Combining fingerprint data with other reconnaissance findings creates a holistic view of the target's infrastructure, aiding in understanding its overall security posture and potential attack vectors

## Fingerprinting Techniques

* `Banner Grabbing`: Banner grabbing involves analysing the banners presented by web servers and other services. These banners often reveal the server software, version numbers, and other details.
* `Analysing HTTP Headers`: HTTP headers transmitted with every web page request and response contain a wealth of information. The `Server` header typically discloses the web server software, while the `X-Powered-By` header might reveal additional technologies like scripting languages or frameworks.
* `Probing for Specific Responses`: Sending specially crafted requests to the target can elicit unique responses that reveal specific technologies or versions. For example, certain error messages or behaviours are characteristic of particular web servers or software components.
* `Analysing Page Content`: A web page's content, including its structure, scripts, and other elements, can often provide clues about the underlying technologies. There may be a copyright header that indicates specific software being used, for example.

A variety of tools exist to automate this process:

| Tool         | Description                                                                                                           | Features                                                                                            |
| ------------ | --------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------- |
| `Wappalyzer` | Browser extension and online service for website technology profiling.                                                | Identifies a wide range of web technologies, including CMSs, frameworks, analytics tools, and more. |
| `BuiltWith`  | Web technology profiler that provides detailed reports on a website's technology stack.                               | Offers both free and paid plans with varying levels of detail.                                      |
| `WhatWeb`    | Command-line tool for website fingerprinting.                                                                         | Uses a vast database of signatures to identify various web technologies.                            |
| `Nmap`       | Versatile network scanner that can be used for various reconnaissance tasks, including service and OS fingerprinting. | Can be used with scripts (NSE) to perform more specialised fingerprinting.                          |
| `Netcraft`   | Offers a range of web security services, including website fingerprinting and security reporting.                     | Provides detailed reports on a website's technology, hosting provider, and security posture.        |
| `wafw00f`    | Command-line tool specifically designed for identifying Web Application Firewalls (WAFs).                             | Helps determine if a WAF is present and, if so, its type and configuration.                         |

## Fingerprinting Process

Getting the headers of a URL:

```bash
curl -I https://<domain>
```

### Wafw00f

Using wafw00f to detect the presence of a Web App Firewall (WAF):

```bash
pip3 install git+https://github.com/EnableSecurity/wafw00f
wafw00f <domain>

                ______
               /      \
              (  W00f! )
               \  ____/
               ,,    __            404 Hack Not Found
           |`-.__   / /                      __     __
           /"  _/  /_/                       \ \   / /
          *===*    /                          \ \_/ /  405 Not Allowed
         /     )__//                           \   /
    /|  /     /---`                        403 Forbidden
    \\/`   \ |                                 / _ \
    `\    /_\\_              502 Bad Gateway  / / \ \  500 Internal Error
      `_____``-`                             /_/   \_\

                        ~ WAFW00F : v2.2.0 ~
        The Web Application Firewall Fingerprinting Toolkit
    
[*] Checking https://<domain>
[+] The site https://<domain> is behind Wordfence (Defiant) WAF.
[~] Number of requests: 2
```

### Nikto

Powerful open-source web server scanner

```bash
sudo apt update && sudo apt install -y perl
git clone https://github.com/sullo/nikto
cd nikto/program
chmod +x ./nikto.pl
```

To scan a domain only running the fingerprinting modules:\\

```bash
nikto -h <domain> -Tuning b
```

The `-Tuning b` flag tells `Nikto` to only run the Software Identification modules.

