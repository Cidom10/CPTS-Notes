# SMNP

The current version is `SNMPv3`

SNMP also transmits control commands using agents over UDP port `161`

SNMP also enables the use of so-called `traps` over UDP port `162`

### MIB

The `Management Information Base` (`MIB`) is a text file in which all queryable SNMP objects of a device are listed in a standardized tree hierarchy. It contains at least one `Object Identifier` (`OID`).

MIB files are written in the `Abstract Syntax Notation One` (`ASN.1`) based ASCII text format.

### OID

An OID represents a node in a hierarchical namespace

Many nodes in the OID tree contain nothing except references to those below them. The OIDs consist of integers and are usually concatenated by dot notation. We can look up many MIBs for the associated OIDs in the [Object Identifier Registry](https://www.alvestrand.no/objectid/).

### SNMPv1

Still used in many small networks

Supports grabbing info from devices, configuration of devices, provides traps which are notifications of events

**No build in authentication** system

Does **not** support encryption

### SNMPv2

Has different versions, today's version is v2c (c means community-based)

The community string that provides security is only transmitted in plain text, so no built-in encryption

### SNMPv3

Now has user/pass auth and transmission encryption

Way more configuration options than v2c

### Community Strings

Passwords used to determine if info can be viewed or not

## Default Configuration

The default configuration of the SNMP daemon defines the basic settings for the service, which include the IP addresses, ports, MIB, OIDs, authentication, and community strings.

Found at `/etc/snmp/snmpd.conf`

Way too much configuration, check the manpage

## Dangerous Settings

| **Settings**                                     | **Description**                                                                       |
| ------------------------------------------------ | ------------------------------------------------------------------------------------- |
| `rwuser noauth`                                  | Provides access to the full OID tree without authentication.                          |
| `rwcommunity <community string> <IPv4 address>`  | Provides access to the full OID tree regardless of where the requests were sent from. |
| `rwcommunity6 <community string> <IPv6 address>` | Same access as with `rwcommunity` with the difference of using IPv6.                  |

## Footprinting the Service

For footprinting SNMP, we can use tools like `snmpwalk`, `onesixtyone`, and `braa`. `Snmpwalk` is used to query the OIDs with their information. `Onesixtyone` can be used to brute-force the names of the community strings since they can be named arbitrarily by the administrator. Since these community strings can be bound to any source, identifying the existing community strings can take quite some time.

SNMPwalk Example

```bash
snmpwalk -v2c -c public <ip>
```

If we do not know the community string, we can use `onesixtyone` and `SecLists` wordlists to identify these community strings.

```bash
sudo apt install onesixtyone
onesixtyone -c /opt/useful/seclists/Discovery/SNMP/snmp.txt <ip>
```

We can use the tool [crunch](https://secf00tprint.github.io/blog/passwords/crunch/advanced/en) to create custom wordlists to guess the community string

Once we have the community string, we can use [braa](https://github.com/mteg/braa) to brute-force the individual OIDs&#x20;

```bash
sudo apt install braa
braa <community string>@<IP>:.1.3.6.*
```
