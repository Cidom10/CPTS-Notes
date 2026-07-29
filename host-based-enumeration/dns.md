# DNS

Several types of DNS Servers:

| **Server Type**                | **Description**                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| ------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `DNS Root Server`              | The root servers of the DNS are responsible for the top-level domains (`TLD`). As the last instance, they are only requested if the name server does not respond. Thus, a root server is a central interface between users and content on the Internet, as it links domain and IP address. The [Internet Corporation for Assigned Names and Numbers](https://www.icann.org/) (`ICANN`) coordinates the work of the root name servers. There are `13` such root servers around the globe. |
| `Authoritative Nameserver`     | Authoritative name servers hold authority for a particular zone. They only answer queries from their area of responsibility, and their information is binding. If an authoritative name server cannot answer a client's query, the root name server takes over at that point. Based on the country, company, etc., authoritative nameservers provide answers to recursive DNS nameservers, assisting in finding the specific web server(s).                                              |
| `Non-authoritative Nameserver` | Non-authoritative name servers are not responsible for a particular DNS zone. Instead, they collect information on specific DNS zones themselves, which is done using recursive or iterative DNS querying.                                                                                                                                                                                                                                                                               |
| `Caching DNS Server`           | Caching DNS servers cache information from other name servers for a specified period. The authoritative name server determines the duration of this storage.                                                                                                                                                                                                                                                                                                                             |
| `Forwarding Server`            | Forwarding servers perform only one function: they forward DNS queries to another DNS server.                                                                                                                                                                                                                                                                                                                                                                                            |
| `Resolver`                     | Resolvers are not authoritative DNS servers but perform name resolution locally in the computer or router.                                                                                                                                                                                                                                                                                                                                                                               |

![Diagram showing domain hierarchy: Root, Top Level Domains (TLD) like net, org, com, dev, io; Second Level Domain inlanefreight.com; Sub-Domains dev.inlanefreight.com, www.inlanefreight.com, mail.inlanefreight.com; Host WS01.dev.inlanefreight.com.](https://academy.hackthebox.com/storage/modules/27/tooldev-dns.png)

Different DNS Records and their uses:

| **DNS Record** | **Description**                                                                                                                                                                                                                                   |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `A`            | Returns an IPv4 address of the requested domain as a result.                                                                                                                                                                                      |
| `AAAA`         | Returns an IPv6 address of the requested domain.                                                                                                                                                                                                  |
| `MX`           | Returns the responsible mail servers as a result.                                                                                                                                                                                                 |
| `NS`           | Returns the DNS servers (nameservers) of the domain.                                                                                                                                                                                              |
| `TXT`          | This record can contain various information. The all-rounder can be used, e.g., to validate the Google Search Console or validate SSL certificates. In addition, SPF and DMARC entries are set to validate mail traffic and protect it from spam. |
| `CNAME`        | This record serves as an alias for another domain name. If you want the domain www.hackthebox.eu to point to the same IP as hackthebox.eu, you would create an A record for hackthebox.eu and a CNAME record for www.hackthebox.eu.               |
| `PTR`          | The PTR record works the other way around (reverse lookup). It converts IP addresses into valid domain names.                                                                                                                                     |
| `SOA`          | Provides information about the corresponding DNS zone and email address of the administrative contact.                                                                                                                                            |

Find person responsible for operation of domain:

```bash
dig soa <domain>
```

## Default Configuration

All servers have 3 different types of configuration files

1. Local DNS config
2. Zone files
3. Reverse name resolution files

The DNS server [Bind9](https://www.isc.org/bind/) is often used on Linux. Local config file is **named.conf**

Find local DNS Configuration:

```bash
cat /etc/bind/named.conf.local
```

A `zone file` is a text file that describes a DNS zone with the BIND file format. In other words it is a point of delegation in the DNS tree.

There must be precisely one `SOA` record and at least one `NS` record. The SOA resource record is usually located at the beginning of a zone file. The main goal of these global rules is to improve the readability of zone files.

Search a zone file:

```bash
cat /etc/bind/db.domain.com
```

For the `Fully Qualified Domain Name` (`FQDN`) to be resolved from the IP address, the DNS server must have a reverse lookup file. In this file, the computer name (`FQDN`) is assigned to the last octet of an IP address, which corresponds to the respective host, using a PTR record. The PTR records are responsible for the reverse translation of IP addresses into names, as we have already seen in the above table.

Search Reverse Name Resolution Zone file:

```bash
cat /etc/bind/db.10.129.14
```

## Dangerous Settings

[List of BIND9 server vulnerabilities](https://www.cvedetails.com/product/144/ISC-Bind.html?vendor_id=64) (CVEDetails)

[SecurityTrails popular DNS server attacks](https://web.archive.org/web/20250329174745/https://securitytrails.com/blog/most-popular-types-dns-attacks)

| **Option**        | **Description**                                                                |
| ----------------- | ------------------------------------------------------------------------------ |
| `allow-query`     | Defines which hosts are allowed to send requests to the DNS server.            |
| `allow-recursion` | Defines which hosts are allowed to send recursive requests to the DNS server.  |
| `allow-transfer`  | Defines which hosts are allowed to receive zone transfers from the DNS server. |
| `zone-statistics` | Collects statistical data of zones.                                            |

## Footprinting the Service

We query using the NS record and specification of our desired server using the `@` character

```bash
dig ns <domain> @<ip>
```

We can also query a DNS server's version using a class CHAOS query and type TXT. It has to exist as a record, though.

```bash
dig CH TXT version.bind <ip>
```

The ANY option can view all available records (that the server is willing to disclose)

```bash
dig any <domain> @<ip>
```

`Zone transfer` refers to the transfer of zones to another server in DNS, which generally happens over TCP port 53. This procedure is abbreviated `Asynchronous Full Transfer Zone` (`AXFR`).

Synchronization between the servers involved is realized by zone transfer. Using a secret key `rndc-key`, which we have seen initially in the default configuration, the servers make sure that they communicate with their own master or slave.

Dig AXFR Zone Transfer:

```bash
dig axfr <domain> @<ip>
```

If the administrator used a subnet for the `allow-transfer` option for testing purposes or as a workaround solution or set it to `any`, everyone would query the entire zone file at the DNS server. In addition, other zones can be queried, which may even show internal IP addresses and hostnames.

Dig Internal AXFR Zone Transfer:

```bash
dig axfr internal.<domain> @<ip>
```

We can brute-force using individual A records. [List of possible hostnames](https://github.com/danielmiessler/SecLists/blob/master/Discovery/DNS/subdomains-top1million-5000.txt) to help send requests in order

```bash
for sub in $(cat /opt/useful/seclists/Discovery/DNS/subdomains-top1million-110000.txt);do dig $sub.<domain> @<ip> | grep -v ';\|SOA' | sed -r '/^\s*$/d' | grep $sub | tee -a subdomains.txt;done
```

[DNSenum](https://github.com/fwaeytens/dnsenum) is a tool that can help with this

```bash
dnsenum --dnsserver <ip> --enum -p 0 -s 0 -o subdomains.txt -f /opt/useful/seclists/Discovery/DNS/subdomains-top1million-110000.txt <domain>
```
