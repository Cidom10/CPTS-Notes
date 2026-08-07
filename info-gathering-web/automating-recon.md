# Automating Recon

Automation offers several key advantages for web reconnaissance:

* `Efficiency`: Automated tools can perform repetitive tasks much faster than humans, freeing up valuable time for analysis and decision-making.
* `Scalability`: Automation allows you to scale your reconnaissance efforts across a large number of targets or domains, uncovering a broader scope of information.
* `Consistency`: Automated tools follow predefined rules and procedures, ensuring consistent and reproducible results and minimising the risk of human error.
* `Comprehensive Coverage`: Automation can be programmed to perform a wide range of reconnaissance tasks, including DNS enumeration, subdomain discovery, web crawling, port scanning, and more, ensuring thorough coverage of potential attack vectors.
* `Integration`: Many automation frameworks allow for easy integration with other tools and platforms, creating a seamless workflow from reconnaissance to vulnerability assessment and exploitation.

## Recon Frameworks

These frameworks aim to provide a complete suite of tools for web reconnaissance:

* [FinalRecon](https://github.com/thewhiteh4t/FinalRecon): A Python-based reconnaissance tool offering a range of modules for different tasks like SSL certificate checking, Whois information gathering, header analysis, and crawling. Its modular structure enables easy customisation for specific needs.
* [Recon-ng](https://github.com/lanmaster53/recon-ng): A powerful framework written in Python that offers a modular structure with various modules for different reconnaissance tasks. It can perform DNS enumeration, subdomain discovery, port scanning, web crawling, and even exploit known vulnerabilities.
* [theHarvester](https://github.com/laramies/theHarvester): Specifically designed for gathering email addresses, subdomains, hosts, employee names, open ports, and banners from different public sources like search engines, PGP key servers, and the SHODAN database. It is a command-line tool written in Python.
* [SpiderFoot](https://github.com/smicallef/spiderfoot): An open-source intelligence automation tool that integrates with various data sources to collect information about a target, including IP addresses, domain names, email addresses, and social media profiles. It can perform DNS lookups, web crawling, port scanning, and more.
* [OSINT Framework](https://osintframework.com/): A collection of various tools and resources for open-source intelligence gathering. It covers a wide range of information sources, including social media, search engines, public records, and more.

## FinalRecon

`FinalRecon` offers a wealth of recon information:

* `Header Information`: Reveals server details, technologies used, and potential security misconfigurations.
* `Whois Lookup`: Uncovers domain registration details, including registrant information and contact details.
* `SSL Certificate Information`: Examines the SSL/TLS certificate for validity, issuer, and other relevant details.
* `Crawler`
* `DNS Enumeration`: Queries over 40 DNS record types, including DMARC records for email security assessment.
* `Subdomain Enumeration`: Leverages multiple data sources (crt.sh, AnubisDB, ThreatMiner, CertSpotter, Facebook API, VirusTotal API, Shodan API, BeVigil API) to discover subdomains.
* `Directory Enumeration`: Supports custom wordlists and file extensions to uncover hidden directories and files.
* `Wayback Machine`: Retrieves URLs from the last five years to analyse website changes and potential vulnerabilities.

```bash
git clone https://github.com/thewhiteh4t/FinalRecon.git
cd FinalRecon
pip3 install -r requirements.txt
chmod +x ./finalrecon.py
./finalrecon.py --help
```
