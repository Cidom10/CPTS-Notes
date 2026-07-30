# IPMI

[Intelligent Platform Management Interface](https://www.thomas-krenn.com/en/wiki/IPMI_Basics) (`IPMI`) is a set of standardized specifications for hardware-based host management systems used for system management and monitoring. It acts as an autonomous subsystem and works independently of the host's BIOS, CPU, firmware, and underlying operating system

IPMI is typically used in three ways:

* Before the OS has booted to modify BIOS settings
* When the host is fully powered down
* Access to a host after a system failure

To function, IPMI requires the following components:

* Baseboard Management Controller (BMC) - A micro-controller and essential component of an IPMI
* Intelligent Chassis Management Bus (ICMB) - An interface that permits communication from one chassis to another
* Intelligent Platform Management Bus (IPMB) - extends the BMC
* IPMI Memory - stores things such as the system event log, repository store data, and more
* Communications Interfaces - local system interfaces, serial and LAN interfaces, ICMB and PCI Management Bus

## Footprinting the Service

Communicates over UDP port 623

Systems that use the IPMI protocol are called Baseboard Management Controllers (BMCs). BMCs are typically implemented as embedded ARM systems running Linux, and connected directly to the host's motherboard

Nmap scan using ipmi-version script to footprint:

```bash
sudo nmap -sU --script ipmi-version -p 623 <ip> (ex: ilo.inlanfreight.local)
```

We can also use the Metasploit scanner module [IPMI Information Discovery (auxiliary/scanner/ipmi/ipmi\_version)](https://www.rapid7.com/db/modules/auxiliary/scanner/ipmi/ipmi_version/).

```
use auxiliary/scanner/ipmi/ipmi_version
```

During internal penetration tests, we often find BMCs where the administrators have not changed the default password. Some unique default passwords to keep in our cheatsheets include:

| Product         | Username      | Password                                                                  |
| --------------- | ------------- | ------------------------------------------------------------------------- |
| Dell iDRAC      | root          | calvin                                                                    |
| HP iLO          | Administrator | randomized 8-character string consisting of numbers and uppercase letters |
| Supermicro IPMI | ADMIN         | ADMIN                                                                     |

## Dangerous Settings

If default credentials do not work to access a BMC, we can turn to a [flaw](https://web.archive.org/web/20260421071724/http://fish2.com/ipmi/remote-pw-cracking.html) in the RAKP protocol in IPMI 2.0. During the authentication process, the server sends a salted SHA1 or MD5 hash of the user's password to the client before authentication takes place

These password hashes can then be cracked offline using a dictionary attack using `Hashcat` mode `7300`. In the event of an HP iLO using a factory default password, we can use this Hashcat mask attack command `hashcat -m 7300 ipmi.txt -a 3 ?1?1?1?1?1?1?1?1 -1 ?d?u` which tries all combinations of upper case letters and numbers for an eight-character password

To retrieve IPMI hashes, we can use the Metasploit [IPMI 2.0 RAKP Remote SHA1 Password Hash Retrieval](https://www.rapid7.com/db/modules/auxiliary/scanner/ipmi/ipmi_dumphashes/) module.

```
use auxiliary/scanner/ipmi/ipmi_dumphashes 
```
