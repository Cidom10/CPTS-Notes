# Domain Information

### Domain Information

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

