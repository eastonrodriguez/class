**Control Number:** 1

**Control Name:** LAN Manager authentication level

**Default State:** 	LmCompatibilityLevel = 1 (“Send LM & NTLM, use NTLMv2 session security if negotiated”) under HKLM:\SYSTEM\CurrentControlSet\Control\Lsa

**Hardened State:** LmCompatibilityLevel = 5 ("Send NTLMv2 response only. Refuse LM & NTLM")

**Implementation Method:** Create a group policy setting that applies to the Servers OU, then force those computers to immediately check for the new policy through gpupdate /force

**Registry/GPO Path:** HKLM:\SYSTEM\CurrentControlSet\Control\Lsa\LmCompatibilityLevel = 5 (DWORD)
GPO: Computer Configuration > Windows Settings > Security Settings > Local Policies > Security Options > Network security: LAN Manager authentication level

**How to Verify:** (Get-ItemProperty 'HKLM:\SYSTEM\CurrentControlSet\Control\Lsa').LmCompatibilityLevel returns 5; gpresult /h report.html shows the policy is applied; Security event 4624 shows NTLM V2, never LM or NTLM V1

**Security Impact:** This change makes the servers stop using old, weak login methods (LM and NTLMv1). Attackers can crack these methods quickly or capture and reuse them to break into other systems. After the change, servers will only accept NTLMv2, which is much harder to attack. This closes a well known hole that attackers use to steal passwords and move from one computer to another inside a network.

**Operational Impact:** Some older devices and software may stop being able to log in to the servers. Things like very old Windows versions (like Windows 95/98 or NT 4.0 without updates), old printers and scanners, old network storage, and some legacy apps. Anything that can only use the old login methods will be refused. Modern Windows systems already support NTLMv2, so most users should not notice a change. 

---

**Control Number:** 2

**Control Name:** Disable SMBv1

**Default State:** Enabled server-wide (Get-SmbServerConfiguration shows EnableSMB1Protocol : True)

**Hardened State:** EnableSMB1Protocol : False and the SMB1 Windows feature removed

**Implementation Method:** Method	Set-SmbServerConfiguration -EnableSMB1Protocol $false -Force Uninstall-WindowsFeature -Name FS-SMB1

**Registry/GPO Path:** HKLM:\SYSTEM\CurrentControlSet\Services\LanmanServer\Parameters\SMB1 = 0 (DWORD)

**How to Verify:** 	(Get-SmbServerConfiguration).EnableSMB1Protocol returns False; (Get-WindowsFeature FS-SMB1).Installed returns; False During the audit step, SMB1 events appear in the Microsoft-Windows-SMBServer/Audit log

**Security Impact:** SMBv1 is an old file-sharing protocol with serious, well-known flaws. Attackers have used it to take over servers without needing a password, and it was the entry point for major attacks like WannaCry and NotPetya, which spread by themselves from computer to computer. Turning it off and removing the feature closes that door. It also means the protocol can't be switched back on by accident later.

**Operational Impact:** Very old devices and software that can only talk SMBv1 will no longer be able to reach file shares on these servers. Devises such as Windows XP and Server 2003, old scanners and copiers that save to a network folder, old network storage devices, and some legacy apps. Modern Windows, Linux, and Mac systems use newer SMB versions and won't be affected.  

---

**Control Number:** 3

**Control Name:** SMB signing (server)

**Default State:** Not required - RequireSecuritySignature = 0 under HKLM:\SYSTEM\CurrentControlSet\Services\LanManServer\Parameters

**Hardened State:** RequireSecuritySignature = 1 (every SMB session must be signed)

**Implementation Method:** Use a GPO, or run Set-SmbServerConfiguration -RequireSecuritySignature $true -Force

**Registry/GPO Path:** HKLM:\SYSTEM\CurrentControlSet\Services\LanManServer\Parameters\RequireSecuritySignature = 1 (DWORD); GPO: Computer Configuration > Windows Settings > Security Settings > Local Policies > Security Options > Microsoft network server: Digitally sign communications

**How to Verify:** (Get-SmbServerConfiguration).RequireSecuritySignature returns True

**Security Impact:** Without signing, an attacker who sits between a client and a server on the network can alter or replay file-sharing traffic without being noticed. They can also relay stolen logins to the server and act as the real user. Requiring signing means every SMB message carries a digital stamp that proves it came from the real sender and wasn't changed along the way. This blocks relay and man-in-the-middle attacks against the servers.

**Operational Impact:** Signing adds a small amount of processing work, so there might be some slightly slower file transfers, mostly on very busy servers or older hardware. Clients that can't sign, such as very old systems, some scanners and copiers, and some Linux or NAS devices that aren't set up for signing, will be refused. Modern Windows clients support signing and should work without changes. 

---

**Control Number:** 4

**Control Name:** LLMNR (Link-Local Multicast Name Resolution)

**Default State:** Enabled - no policy configured under HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\DNSClient

**Hardened State:** EnableMulticast = 0 (LLMNR off)

**Implementation Method:** GPO linked to the servers OU

**Registry/GPO Path:** HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\DNSClient\EnableMulticast = 0 (DWORD); GPO: Computer Configuration > Administrative Templates > Network > DNS Client > Turn off multicast name resolution = Enabled

**How to Verify:** (Get-ItemProperty 'HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\DNSClient').EnableMulticast returns 0

**Security Impact:** When a computer can't find another computer by name through DNS, LLMNR makes it shout the question to everyone on the local network. An attacker on the same network can answer "that's me!" and trick the computer into sending its login information. The attacker can then crack the password or reuse the login to get into other systems. Tools like Responder do this automatically. Turning LLMNR off removes this easy way to steal logins.

**Operational Impact:** Name lookups now depend fully on DNS, so any name DNS can't resolve will simply fail instead of falling back to the local network shout. Problems are most likely in small or unmanaged networks, with old devices, and with systems that rely on short names that aren't in DNS. In a normal domain with healthy DNS, most users won't notice a change.

---

**Control Number:** 6

**Control Name:** NTLM traffic restriction

**Default State:** Not configured - RestrictSendingNTLMTraffic absent (equivalent to “Allow all”)

**Hardened State:** 	Phase 1: RestrictSendingNTLMTraffic = 1 (audit all) for 2-4 weeks; Phase 2: RestrictSendingNTLMTraffic = 2 (deny all), with a written exception list in ClientAllowedNTLMServers

**Implementation Method:** Link a GPO to the Servers OU. Audit outgoing NTLM for 2-4 weeks, fix or list exceptions, then switch to Deny all and run gpupdate /force.

**Registry/GPO Path:** HKLM:\SYSTEM\CurrentControlSet\Control\Lsa\MSV1_0\RestrictSendingNTLMTraffic; GPO: Computer Configuration > Windows Settings > Security Settings > Local Policies > Security Options > Network security: Restrict NTLM: Outgoing NTLM traffic to remote servers

**How to Verify:** (Get-ItemProperty 'HKLM:\SYSTEM\CurrentControlSet\Control\Lsa\MSV1_0').RestrictSendingNTLMTraffic

**Security Impact:** NTLM is an old login method that can be stolen and reused. If an attacker tricks a server into sending an NTLM login to a fake or hostile machine, they can crack the password or reuse the login to get into other systems. Blocking outgoing NTLM forces servers to use Kerberos, which is much harder to attack. It also lowers the damage if an attacker is already inside the network, because the servers can no longer be pushed into handing out logins this way.

**Operational Impact:** This is one of the riskier controls. Anything on the servers that still logs in to other machines with NTLM will fail once "Deny all" is on. Common causes are connecting by IP address instead of name, missing or duplicate SPNs, old apps, and old file servers or devices. 

---

**Control Number:** 8

**Control Name:** Deploy Windows LAPS

**Default State:** 	No LAPS.

**Hardened State:** Windows LAPS manages the built-in Administrator account. Each server gets a unique random 20-character password, changed every 30 days and stored encrypted in Active Directory.

**Implementation Method:** Update-LapsADSchema (one time, needs a schema admin); Set-LapsADComputerSelfPermission -Identity "<servers OU>"; Use a GPO to set the policy, and Set-LapsADReadPasswordPermission to let the help desk group read passwords

**Registry/GPO Path:** HKLM:\SOFTWARE\Policies\Microsoft\Windows\LAPS values BackupDirectory = 2 (AD), PasswordAgeDays = 30, PasswordLength = 20, PasswordComplexity = 4; GPO: Computer Configuration > Administrative Templates > System > LAPS

**How to Verify:** Get-LapsADPassword -Identity prod-fs-02 -AsPlainText returns a password and an expiration time; Get-WinEvent -LogName Microsoft-Windows-LAPS/Operational shows password updates succeeding; Check that a second server has a different password

**Security Impact:** Without LAPS, many servers often share the same local Administrator password. If an attacker steals that password from one server, they can log in to every other server with it. LAPS gives each server its own random password and changes it on a schedule. A stolen password only works on one server and only for a short time. The passwords are stored encrypted in Active Directory, and only approved groups can read them.

**Operational Impact:** Admins can no longer type a remembered shared password. They must look up the current password for each server first, so the help desk needs training and access to the right tool. The one-time schema update needs a schema admin and affects the whole forest, so plan it carefully. If a server loses contact with Active Directory, or the permissions are wrong, the password may not be saved, and you could be locked out of the local Administrator account.

---

**Control Number:** 9

**Control Name:** Enable Credential Guard

**Default State:** Not enabled. No LsaCfgFlags is set and VBS is off

**Hardened State:** VBS on with Secure Boot, and LsaCfgFlags = 1

**Implementation Method:** GPO, then reboot. 

**Registry/GPO Path:** HKLM:\SYSTEM\CurrentControlSet\Control\DeviceGuard\EnableVirtualizationBasedSecurity = 1
HKLM:\SYSTEM\CurrentControlSet\Control\DeviceGuard\RequirePlatformSecurityFeatures = 1
HKLM:\SYSTEM\CurrentControlSet\Control\Lsa\LsaCfgFlags = 1; GPO: Computer Configuration > Administrative Templates > System > Device Guard > Turn On Virtualization Based Security

**How to Verify:** (Get-CimInstance -ClassName Win32_DeviceGuard -Namespace root\Microsoft\Windows\DeviceGuard).SecurityServicesRunning includes 1
msinfo32 lists Credential Guard under "Virtualization-based security Services Running"

**Security Impact:** When someone logs in to a Windows server, their password hashes and Kerberos tickets are kept in memory. Attackers with admin access use tools like Mimikatz to pull these out of memory and reuse them to move to other systems (things like pass-the-hash and pass-the-ticket). Credential Guard moves these secrets into a protected area that is isolated from the normal operating system, so even an attacker with full admin rights on the server can't read them. This makes it much harder to steal logins from a server that has been compromised.

**Operational Impact:** Credential Guard needs hardware and firmware support such as64-bit CPU with virtualization, UEFI, Secure Boot, and for virtual machines, and support from the hypervisor. Servers that don't meet this will not turn it on. It needs a reboot, and it uses a small amount of extra memory and CPU. NTLMv1, unconstrained Kerberos delegation, and some older apps stop working once it is on. It also blocks some older VPN and Wi-Fi setups from using saved credentials.

---

**Control Number:** 11

**Control Name:** 	Advanced Audit Policy - Logon/Logoff subcategories

**Default State:** Only legacy “Basic” auditing configured via auditpol; Advanced Audit Policy subcategories not enabled

**Hardened State:** Advanced Audit Policy is turned on for the Logon/Logoff subcategories, and legacy audit settings are ignored (SCENoApplyLegacyAuditPolicy = 1)

**Implementation Method:** Create a GPO for the Servers OU. Under Computer Configuration > Windows Settings > Security Settings > Local Policies > Security Options, set "Audit: Force audit policy subcategory settings to override audit policy category settings" to Enabled. Under Advanced Audit Policy Configuration > Audit Policies > Logon/Logoff, set the subcategories in the table above. Then run gpupdate /force

**Registry/GPO Path:** HKLM:\SYSTEM\CurrentControlSet\Control\Lsa\SCENoApplyLegacyAuditPolicy = 1 (DWORD); GPO: Computer Configuration > Windows Settings > Security Settings > Advanced Audit Policy Configuration > Audit Policies > Logon/Logoff

**How to Verify:** auditpol /get /category:"Logon/Logoff" shows the expected settings; Sign in and out, then check that events 4624, 4625, 4634, and 4672 appear in the Security log

**Security Impact:** Logon and logoff events show who signed in to a server, when, from where, and whether they failed. Without them, you can't see password guessing, stolen logins being reused, or an attacker moving between servers. Detailed logon auditing gives the security team the trail they need to spot attacks early and to investigate after one. It also helps meet audit and compliance rules.

**Operational Impact:** Logon auditing creates a lot of events, especially on busy servers such as file servers and domain controllers. The Security log can fill up and overwrite older events quickly, and log storage or SIEM costs may go up. 

---

**Control Number:** 12

**Control Name:** Enable PowerShell Script Block Logging

**Default State:** 	Disabled: EnableScriptBlockLogging = 0

**Hardened State:** EnableScriptBlockLogging = 1

**Implementation Method:** Create a GPO for the Servers OU. Set "Turn on PowerShell Script Block Logging" to Enabled. Leave "Log script block invocation start / stop events" unchecked. Run gpupdate /force on the servers. 

**Registry/GPO Path:** HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging\EnableScriptBlockLogging = 1 (DWORD); GPO: Computer Configuration > Administrative Templates > Windows Components > Windows PowerShell > Turn on PowerShell Script Block Logging

**How to Verify:** Run a test command, then Get-WinEvent -FilterHashtable @{LogName='Microsoft-Windows-PowerShell/Operational'; Id=4104} shows the script text

**Security Impact:** Attackers often use PowerShell because it is built in and can run in memory without leaving files behind. Script block logging records the actual code that PowerShell runs, even when the attacker hides it with encoding or tricks to make it hard to read. PowerShell decodes it before running, and the log captures the decoded version. This gives the security team a clear record of what was run on a server, which helps them spot attacks and investigate after one.

**Operational Impact:** This creates more events in the PowerShell log, especially on servers that run many scripts, such as ones with scheduled tasks or monitoring tools. The log can fill up and overwrite older events, so increase the size of the Microsoft-Windows-PowerShell/Operational log and consider sending it to a central collector. Performance impact is usually small. Scripts may contain passwords or other secrets typed in as plain text, and these will be saved in the log, so people should limit who can read it.

---

**Control Number:** 13

**Control Name:** Command-line process auditing

**Default State:** ProcessCreationIncludeCmdLine_Enabled = 0

**Hardened State:** ProcessCreationIncludeCmdLine_Enabled = 1

**Implementation Method:** Create a GPO for the Servers OU. Under Computer Configuration > Windows Settings > Security Settings > Advanced Audit Policy Configuration > Audit Policies > Detailed Tracking, set "Audit Process Creation" to Success. Under Computer Configuration > Administrative Templates > System > Audit Process Creation, set "Include command line in process creation events" to Enabled. Run gpupdate /force

**Registry/GPO Path:** HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System\Audit\ProcessCreationIncludeCmdLine_Enabled = 1 (DWORD); GPO: Computer Configuration > Administrative Templates > System > Audit Process Creation > Include command line in process creation events Advanced Audit Policy > Detailed Tracking > Audit Process Creation. 

**How to Verify:** 	Run whoami, then find event 4688 in the Security log and check that "Process Command Line" is filled in auditpol /get /subcategory:"Process Creation" shows Success

**Security Impact:** Attackers run many tools and commands on a server after they get in. Without command lines, the log only says that a program started. It doesn't say what it was told to do. With command lines, the security team can see the exact commands, such as ones that create new admin accounts, turn off security tools, or download malware. This makes it much easier to spot attacks and to work out what happened afterward.

**Operational Impact:** This creates a lot of events, because every program that starts is logged. Busy servers can fill the Security log quickly, and log storage or SIEM costs may go up. Command lines can also contain passwords or other secrets that someone typed into a command, and these will be saved in the log in plain text. Limit who can read the Security log and the collected copies. Before rollout, increase the Security log size or send logs to a central collector. Performance impact is usually small.

---

**Control Number:** 15

**Control Name:** Object access auditing on sensitive shares

**Default State:** No SACL configured on C:\FinanceShare - no audit trail of who reads/writes files there

**Hardened State:** The Audit File System subcategory is on (Success and Failure), and a SACL on C:\FinanceShare records changes to files and permissions by Everyone. Both parts are needed. The SACL does nothing unless the subcategory is on, and the subcategory logs nothing unless a SACL exists.

**Implementation Method:** Turn on Object Access > Audit File System with a GPO, then set the SACL:
$acl = Get-Acl C:\FinanceShare
$rule = New-Object System.Security.AccessControl.FileSystemAuditRule('Everyone','Modify,Delete,ChangePermissions,TakeOwnership','ContainerInherit,ObjectInherit','None','Success,Failure')
$acl.AddAuditRule($rule); Set-Acl C:\FinanceShare $acl

**Registry/GPO Path:** GPO: Computer Configuration > Windows Settings > Security Settings > Advanced Audit Policy Configuration > Audit Policies > Object Access > Audit File System
There is no registry key for this. The SACL is stored on the NTFS folder itself.

**How to Verify:** (Get-Acl C:\FinanceShare -Audit).Audit lists the rule; Create, edit, and delete a test file, then look for events 4663 (access attempt), 4660 (delete), and 4670 (permissions changed) in the Security log

**Security Impact:** Finance files are a common target for theft, tampering, and ransomware. Without auditing, you can't tell who opened, changed, deleted, or re-permissioned a file, or when. With a SACL in place, the log shows the account, the file, and the time. This helps the security team spot data theft, unusual bulk changes, and permission changes early. It also gives clear evidence for investigations and audits.

**Operational Impact:** Every audited file action creates an event, so a busy share can fill the Security log quickly, and log storage or SIEM costs may go up. Auditing "Everyone" with Success and Failure can be very noisy, so limit it to the actions that matter. Before rollout, increase the Security log size or send logs to a central collector. Performance impact is usually small on a single folder but grows if you audit many shares.

---

**Control Number:** 16

**Control Name:** AppLocker

**Default State:** Not configured - no rules defined, service not running

**Hardened State:** AppLocker is on with default rules for Executable, Windows Installer, Script, and Packaged app rules. These rules allow only programs from Windows and Program Files, plus administrators. It starts in Audit only mode, then moves to Enforce after review. The Application Identity service (AppIDSvc) is set to Automatic and running.

**Implementation Method:** Create a GPO for the Servers OU. Under Application Control Policies > AppLocker, right-click each rule type and choose Create Default Rules. Set each rule type to Audit only (AppLocker Properties > Enforcement). In the same GPO, set the Application Identity service to Automatic (Computer Configuration > Windows Settings > Security Settings > System Services). Run gpupdate /force 

**Registry/GPO Path:** GPO: Computer Configuration > Windows Settings > Security Settings > Application Control Policies > AppLocker; Rules are stored under HKLM:\SOFTWARE\Policies\Microsoft\Windows\SrpV2

**How to Verify:** (Get-Service AppIDSvc).Status returns Running Get-AppLockerPolicy -Effective -Xml shows the rules; Use Test-AppLockerPolicy on a sample file; Events 8003 (audit) and 8004 (blocked) in Microsoft-Windows-AppLocker/EXE and DLL

**Security Impact:** Attackers often run their own programs, scripts, or installers on a server after they get in, such as ransomware, hacking tools, or remote access tools. AppLocker only lets approved programs run, so unknown software is blocked even if the attacker gets it onto the server. This stops many common attacks and limits the damage if a server is compromised.

**Operational Impact:** This is one of the higher-effort controls. If the rules are too strict, legitimate programs, scripts, and installers can be blocked, including admin tools, backup agents, monitoring tools, and software updates. 

---

**Control Number:** 18

**Control Name:** PowerShell language mode

**Default State:** FullLanguage

**Hardened State:** ConstrainedLanguage

**Implementation Method:** Enforce AppLocker Script rules (or use WDAC). The __PSLockdownPolicy environment variable is for testing only, because it is not a real security boundary.

**Registry/GPO Path:** GPO: AppLocker > Script Rules set to Enforce; Rules are stored under HKLM:\SOFTWARE\Policies\Microsoft\Windows\SrpV2\Script

**How to Verify:** $ExecutionContext.SessionState.LanguageMode returns ConstrainedLanguage in a normal session; Check that an approved admin script still runs and that a test use of Add-Type is blocked

**Security Impact:** In the default mode, PowerShell can use the full power of Windows. Attackers use this to run tricks such as loading hidden code into memory, calling Windows functions directly, and using .NET features to download and run malware without leaving files behind. Constrained Language Mode takes these features away for normal scripts and sessions. Only approved scripts run with full power. This stops many common attack tools and makes PowerShell much less useful to an attacker who gets onto a server.

**Operational Impact:** Scripts that use blocked features will fail or behave differently. This includes many scripts that use Add-Type, create certain .NET or COM objects, or call .NET methods directly. Admins also lose these features when typing commands by hand in a normal session. Third-party tools, monitoring agents, backup software, and older in-house scripts may break.

---

**Control Number:** 19

**Control Name:** Attack Surface Reduction rules

**Default State:** None enabled

**Hardened State:** Selected Attack Surface Reduction (ASR) rules are turned on.

**Implementation Method:** Add-MpPreference -AttackSurfaceReductionRules_Ids <GUID> -AttackSurfaceReductionRules_Actions AuditMode, then change to Enabled after the audit period. Defender Antivirus must be the main antivirus with real-time protection on.

**Registry/GPO Path:** HKLM:\SOFTWARE\Policies\Microsoft\Windows Defender\Windows Defender Exploit Guard\ASR\Rules; GPO: Computer Configuration > Administrative Templates > Windows Components > Microsoft Defender Antivirus > Microsoft Defender Exploit Guard > Attack Surface Reduction > Configure Attack Surface Reduction rules

**How to Verify:** (Get-MpPreference).AttackSurfaceReductionRules_Ids and .AttackSurfaceReductionRules_Actions list the rules and their modes; Events 1122 (audit) and 1121 (block) in Microsoft-Windows-Windows Defender/Operational

**Security Impact:** After getting in, attackers often steal passwords from memory, run commands on other machines remotely, hide in Windows management features, or run malicious files from email. ASR rules block these common behaviors before they do damage.

**Operational Impact:** Rules in Block mode can stop legitimate programs, such as admin tools, management software, backup agents, and older apps. The PSExec and WMI rule is a common cause of problems, because it can break remote management tools such as Configuration Manager.

---

**Control Number:** 20

**Control Name:** Require Network Level Authentication for RDP

**Default State:** Not required - UserAuthentication = 0 under HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp

**Hardened State:** UserAuthentication = 1

**Implementation Method:** Network Level Authentication (NLA) is required for all RDP connections (UserAuthentication = 1). Users must prove who they are before a remote session is created on the server.

**Registry/GPO Path:** HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp\UserAuthentication = 1 (DWORD); GPO: Computer Configuration > Administrative Templates > Windows Components > Remote Desktop Services > Remote Desktop Session Host > Security > Require user authentication for remote connections by using Network Level Authentication

**How to Verify:** (Get-CimInstance -Namespace root\cimv2\TerminalServices -ClassName Win32_TSGeneralSetting).UserAuthenticationRequired returns 1; Try to connect from a client that does not support NLA and confirm it is refused

**Security Impact:** Without NLA, anyone who can reach the server over the network gets a full Windows login screen before proving who they are. This lets attackers guess passwords, and it uses up server memory and CPU for each connection, which can be used to slow or crash the server. NLA makes the client log in first, so unauthenticated attackers never get that far. It also protects against some RDP flaws that can be used before login, and it makes it harder to steal credentials in a man-in-the-middle attack.

**Operational Impact:** Very old RDP clients that can't do NLA will be refused. Modern Windows, Mac, and most current clients work fine. Users also can't use the option to change an expired password at the RDP login screen the same way, so they may need to change it another way. The client computer must be able to reach a domain controller to check logins, which can cause problems for computers that are not joined to the domain.

---

