# NIX Shells

## Infiltrating Unix/Linux

W3Techs maintains an ongoing OS usage statistics [study](https://w3techs.com/technologies/overview/operating_system). This study reports that over `70%` of websites (webservers) run on a Unix-based system

### Gaining a Shell Through Attacking a Vulnerable Application

As in most engagements, we will start with an initial enumeration of the system using `Nmap`.

What information could we gather from the output?

Find an opening, such as a local web application running something like rConfig.&#x20;

Find version numbers and look for CVEs, exploits, Proof of Concepts, etc

Can also search in metasploit. The example finds a metasploit exploit for the vulnerable version of rConfig

### Spawning a TTY Shell with Python

When we drop into the system shell, we notice that no prompt is present, yet we can still issue some system commands. This is a shell typically referred to as a non-tty shell

We can manually spawn a TTY shell using Python if it is present on the system. We can always check for Python's presence on Linux systems by typing the command: `which python`. To spawn the TTY shell session using Python, we type the following command:

```bash
python -c 'import pty; pty.spawn("/bin/sh")'
```

## Spawning Interactive Shells

There may be times that we land on a system with a limited shell, and Python is not installed. In these cases, it's good to know that we could use several different methods to spawn an interactive shell. Let's examine some of them.

bin/sh -i

```
/bin/sh -i
```

Perl

```
perl -e 'exec "/bin/sh";'
```

Ruby (Run from a script)

```
ruby: exec "/bin/sh"
```

Lua (Run from a script)

```
lua: os.execute('/bin/sh')
```

AWK

```
awk 'BEGIN {system("/bin/sh")}'
```

Find

```
find / -name nameoffile -exec /bin/awk 'BEGIN{system(".bin/sh")}' \;
```

Exec

```
find . -exec /bin/sh \; -quit
```

VIM

```
vim -c ':!/bin/sh'
```

```vim
vim
:set shell=/bin/sh
:shell
```

In addition to knowing about all the options listed above, we should be mindful of the permissions we have with the shell session's account. We can always attempt to run this command to list the file properties and permissions our account has over any given file or binary:

```
ls -la <path>
```



