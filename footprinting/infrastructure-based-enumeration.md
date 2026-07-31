# Infrastructure-Based Enumeration

## Domain Information

```
// Some code
```

To start passive info gathering, begin with scrutinizing their main website

Pay attention to what you see and do not see

Take a developer's view on their offered services and what that could entail

### Online Presence

First point of presence is SSL Certificates

Find subdomains with [crt.sh](https://crt.sh). This source is [Certificate Transparency](https://en.wikipedia.org/wiki/Certificate_Transparency) logs.

To get JSON format of crt.sh:

```bash
curl -s https://crt.sh/\?q\=<Target Domain>\&output\=json | jq .
```

To filter by unique subdomain:

```bash
curl -s https://crt.sh/\?q\=<Target Domain>\&output\=json | jq . | grep name | cut -d":" -f2 | grep -v "CN=" | cut -d'"' -f2 | awk '{gsub(/\\n/,"\n");}1;' | sort -u
```

Identify hosts accessible from internet, not hosted by third-party providers

```bash
for i in $(cat subdomainlist);do host $i | grep "has address" | grep <Target Domain> | cut -d" " -f1,4;done
```

Use [Shodan](https://www.shodan.io/) to find devices and systems permanently connected to the internet. Searches for open TCP/IP ports

Search Shodan for specified IPs:

```bash
for i in $(cat subdomainlist);do host $i | grep "has address" | grep inlanefreight.com | cut -d" " -f4 >> ip-addresses.txt;done
$ for i in $(cat ip-addresses.txt);do shodan host $i;done
```

Use **dig** to display available DNS records:

```bash
dig any <Target>
```

## Cloud Resources

Great place to start with cloud is S3 buckets (AWS), blobs (Azure), and cloud storage (GCP)

Using Google Dork to search for open cloud storage:

```
intext:<Company Name> inurl:amazonaws.com
intext:<Company Name> inurl:blob.core.windows.net
```



[domain.glass](https://domain.glass) is a third-party tool to tell a lot about a company's infrastructure.

[GreyHatWarfare](https://buckets.grayhatwarfare.com/) has different searches, cloud storage discovery, and filters for file formats

## Staff

Searching for and identifying employees on social media platforms can also reveal a lot about the teams' infrastructure and makeup. This, in turn, can lead to us identifying which technologies, programming languages, and even software applications are being used. To a large extent, we will also be able to assess each person's focus based on their skills. The posts and material shared with others are also a great indicator of what the person is currently engaged in and what that person currently feels is important to share with others.



Job postings can give clues to what their infrastructure holds.



LinkedIn offers large amounts of searches and filtering to get info on staff and a company's infrastructure
