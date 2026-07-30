# Windows RMPs

Windows servers can be managed locally using Server Manager administration tasks on remote servers. Remote management is enabled by default starting with Windows Server 2016

The main components used for remote management of Windows and Windows servers are the following:

* Remote Desktop Protocol (`RDP`)
* Windows Remote Management (`WinRM`)
* Windows Management Instrumentation (`WMI`)

## RDP

RDP works at the application layer in the TCP/IP reference model, typically utilizing TCP port 3389 as the transport protocol. However, the connectionless UDP protocol can use port 3389 also for remote administration

For an RDP session to be established, both the network firewall and the firewall on the server must allow connections from the outside.&#x20;

If [Network Address Translation](https://en.wikipedia.org/wiki/Network_address_translation) (`NAT`) is used on the route between client and server, as is often the case with Internet connections, the remote computer needs the public IP address to reach the server. In addition, port forwarding must be set up on the NAT router in the direction of the server

Many Windows systems do not insist on this but still accept inadequate encryption via [RDP Security](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-rdpbcgr/8e8b2cca-c1fa-456c-8ecb-a82fc60b2322). Nevertheless, even with this, an attacker is still far from being locked out because the identity-providing certificates are merely self-signed by default

The `Remote Desktop` service is installed by default on Windows servers and does not require additional external applications. This service can be activated using the `Server Manager` and comes with the default setting to allow connections to the service only to hosts with [Network level authentication](https://en.wikipedia.org/wiki/Network_Level_Authentication) (`NLA`).

### Footprinting the Service

Scanning the RDP service can quickly give us a lot of information about the host. For example, we can determine if `NLA` is enabled on the server or not, the product version, and the hostname

```bash
nmap -sV -sC <ip> -p3389 --script rdp*
```

In addition, we can use `--packet-trace` to track the individual packages and inspect their contents manually. We can see that the `RDP cookies` (`mstshash=nmap`) used by Nmap to interact with the RDP server can be identified by `threat hunters` and various security services such as [Endpoint Detection and Response](https://en.wikipedia.org/wiki/Endpoint_detection_and_response) (`EDR`), and can lock us out as penetration testers on hardened networks

```bash
nmap -sV -sC 10.129.201.248 -p3389 --packet-trace --disable-arp-ping -n
```

A Perl script named [rdp-sec-check.pl](https://github.com/CiscoCXSecurity/rdp-sec-check) has also been developed by [Cisco CX Security Labs](https://github.com/CiscoCXSecurity) that can unauthentically identify the security settings of RDP servers based on the handshakes

RDP Security Check

```bash
sudo cpan
git clone https://github.com/CiscoCXSecurity/rdp-sec-check.git && cd rdp-sec-check
./rdp-sec-check.pl 10.129.201.248
```

We can connect to RDP servers on Linux using `xfreerdp`, `rdesktop`, or `Remmina` and interact with the GUI of the server accordingly

```bash
xfreerdp /u:cry0l1t3 /p:"P455w0rd!" /v:<ip>
```

## WinRM

The Windows Remote Management (`WinRM`) is a simple Windows integrated remote management protocol based on the command line.&#x20;

WinRM uses the Simple Object Access Protocol (`SOAP`) to establish connections to remote hosts and their applications. Therefore, WinRM must be explicitly enabled and configured starting with Windows 10.&#x20;

WinRM relies on `TCP` ports `5985` and `5986` for communication, with the last port `5986 using HTTPS`, as ports 80 and 443 were previously used for this task. However, since port 80 was mainly blocked for security reasons, the newer ports 5985 and 5986 are used today

### Footprinting the Service

Often we will see that only HTTP (`TCP 5985`) is used instead of HTTPS (`TCP 5986`)

```bash
nmap -sV -sC <ip> -p5985,5986 --disable-arp-ping -n
```

If we want to find out whether one or more remote servers can be reached via WinRM, we can easily do this with the help of PowerShell. The [Test-WsMan](https://docs.microsoft.com/en-us/powershell/module/microsoft.wsman.management/test-wsman?view=powershell-7.2) cmdlet is responsible for this, and the host's name in question is passed to it. In Linux-based environments, we can use the tool called [evil-winrm](https://github.com/Hackplayers/evil-winrm), another penetration testing tool designed to interact with WinRM.

```bash
evil-winrm -i <ip> -u Cry0l1t3 -p P455w0rD!
```

## WMI

Windows Management Instrumentation (`WMI`) is Microsoft's implementation and also an extension of the Common Information Model (`CIM`), core functionality of the standardized Web-Based Enterprise Management (`WBEM`) for the Windows platform

### Footprinting the Service

The initialization of the WMI communication always takes place on `TCP` port `135`, and after the successful establishment of the connection, the communication is moved to a random port. For example, the program [wmiexec.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/wmiexec.py) from the Impacket toolkit can be used for this

```bash
.../wmiexec.py Cry0l1t3:"P455w0rD!"@<ip> "hostname"
```

