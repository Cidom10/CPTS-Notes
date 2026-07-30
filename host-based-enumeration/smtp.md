# SMTP

Defaults to port 25, newer servers also use ports such as TCP 587

Unencrypted, sends all commanda data, and auth info in plain text. Used with SSL/TLS to prevent unauthorized reading

Most modern servers use ESMTP with SMTP-Auth

The SMTP client is known as the Mail User Agent (MUA)

When MUA converts into header and body, the Mail Transfer Agent (MTA) checks email and stores it

A Mail Submission Agent (MSA) (aka Relay server) Checks the validity/origin of email.

<figure><img src="../.gitbook/assets/Screenshot 2026-07-29 at 12.06.47 PM.png" alt=""><figcaption></figcaption></figure>

## Default Configuration

Get default config:

```bash
cat /etc/postfix/main.cf 
```

Sending and communication are done by special commands

| **Command**  | **Description**                                                                                  |
| ------------ | ------------------------------------------------------------------------------------------------ |
| `AUTH PLAIN` | AUTH is a service extension used to authenticate the client.                                     |
| `HELO`       | The client logs in with its computer name and thus starts the session.                           |
| `MAIL FROM`  | The client names the email sender.                                                               |
| `RCPT TO`    | The client names the email recipient.                                                            |
| `DATA`       | The client initiates the transmission of the email.                                              |
| `RSET`       | The client aborts the initiated transmission but keeps the connection between client and server. |
| `VRFY`       | The client checks if a mailbox is available for message transfer.                                |
| `EXPN`       | The client also checks if a mailbox is available for messaging with this command.                |
| `NOOP`       | The client requests a response from the server to prevent disconnection due to time-out.         |
| `QUIT`       | The client terminates the session.                                                               |

We can use telnet to initialize a TCP connection with a SMTP server. From there, we can run the commands above

```bash
telnet <ip> 25
```

The VRFY command can enumerate existing users on the server, but does not always work depending on the server's configuration

If you have to work through a web proxy, the way to get the proxy to connect to the SMTP server would look like `CONNECT <ip>:25 HTTP/1.0`

## Dangerous Settings

**Open Relay Configuration**

```shell-session
mynetworks = 0.0.0.0/0
```

With this setting, this SMTP server can send fake emails and thus initialize communication between multiple parties. Another attack possibility would be to spoof the email and read it.

## Footprinting the Service

Nmap has a `smtp-commands` script that usees EHL0 command

```bash
sudo nmap <ip> -sC -sV -p25
```

We can use the `smtp-open-relay` script to identify the server as an open relay

```bash
sudo nmap <ip> -p25 --script smtp-open-relay -v
```
