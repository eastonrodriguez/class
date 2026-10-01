**Control Number:** 1

**Control Name:** LAN Manager authentication

**Default State:** 	LmCompatibilityLevel = 1 

**Hardened State:** LmCompatibilityLevel = 5

**Implementation Method:** Create a group policy setting that applies to the Servers OU, then force those computers to immediately check for the new policy through gpupdate /force

**Registry/GPO Path:** HKLM:\SYSTEM\CurrentControlSet\Control\Lsa\LmCompatibilityLevel = 5 (DWORD)
GPO: Computer Configuration > Windows Settings > Security Settings > Local Policies > Security Options > Network security: LAN Manager authentication level

**How to Verify:** I

**Security Impact:** I

**Operational Impact:** I

---

**Control Number:** 2

**Control Name:** Disable SMBv1

**Default State:** Enabled: Get-SmbServerConfiguration shows EnableSMB1Protocol : True

**Hardened State:** EnableSMB1Protocol : False and the SMB1 Windows feature removed

**Implementation Method:** Method	Set-SmbServerConfiguration -EnableSMB1Protocol $false -Force Uninstall-WindowsFeature -Name FS-SMB1

**Registry/GPO Path:** HKLM:\SYSTEM\CurrentControlSet\Services\LanmanServer\Parameters\SMB1 = 0 (DWORD)

**How to Verify:** 	(Get-SmbServerConfiguration).EnableSMB1Protocol returns False; (Get-WindowsFeature FS-SMB1).Installed returns; False During the audit step, SMB1 events appear in the Microsoft-Windows-SMBServer/Audit log

**Security Impact:** I

**Operational Impact:** I

---

**Control Number:** 3

**Control Name:** Require SMB server signing

**Default State:** Not required: RequireSecuritySignature = 0 

**Hardened State:** RequireSecuritySignature = 1

**Implementation Method:** Use a GPO, or run Set-SmbServerConfiguration -RequireSecuritySignature $true -Force

**Registry/GPO Path:** HKLM:\SYSTEM\CurrentControlSet\Services\LanManServer\Parameters\RequireSecuritySignature = 1 (DWORD); GPO: Computer Configuration > Windows Settings > Security Settings > Local Policies > Security Options > Microsoft network server: Digitally sign communications

**How to Verify:** (Get-SmbServerConfiguration).RequireSecuritySignature returns True

**Security Impact:** I

**Operational Impact:** I

---

**Control Number:** 4

**Control Name:** Disable LLMNR

**Default State:** 	Enabled.

**Hardened State:** EnableMulticast = 0

**Implementation Method:** GPO linked to the servers OU

**Registry/GPO Path:** HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\DNSClient\EnableMulticast = 0 (DWORD); GPO: Computer Configuration > Administrative Templates > Network > DNS Client > Turn off multicast name resolution = Enabled

**How to Verify:** (Get-ItemProperty 'HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\DNSClient').EnableMulticast returns 0

**Security Impact:** I

**Operational Impact:** I

---

**Control Number:** 6

**Control Name:** Restrict outgoing NTLM traffic

**Default State:** Not configured: RestrictSendingNTLMTraffic is absent (same as "Allow all")

**Hardened State:** 	Phase 1: RestrictSendingNTLMTraffic = 1 (audit all) for 2-4 weeks; Phase 2: RestrictSendingNTLMTraffic = 2 (deny all), with a written exception list in ClientAllowedNTLMServers

**Implementation Method:** I

**Registry/GPO Path:** HKLM:\SYSTEM\CurrentControlSet\Control\Lsa\MSV1_0\RestrictSendingNTLMTraffic; GPO: Computer Configuration > Windows Settings > Security Settings > Local Policies > Security Options > Network security: Restrict NTLM: Outgoing NTLM traffic to remote servers

**How to Verify:** (Get-ItemProperty 'HKLM:\SYSTEM\CurrentControlSet\Control\Lsa\MSV1_0').RestrictSendingNTLMTraffic

**Security Impact:** I

**Operational Impact:** I

---

**Control Number:** 8

**Control Name:** Deploy Windows LAPS

**Default State:** 	No LAPS.

**Hardened State:** Windows LAPS manages the built-in Administrator account. Each server gets a unique random 20-character password, changed every 30 days and stored encrypted in Active Directory.

**Implementation Method:** Update-LapsADSchema (one time, needs a schema admin); Set-LapsADComputerSelfPermission -Identity "<servers OU>"; Use a GPO to set the policy, and Set-LapsADReadPasswordPermission to let the help desk group read passwords

**Registry/GPO Path:** HKLM:\SOFTWARE\Policies\Microsoft\Windows\LAPS values BackupDirectory = 2 (AD), PasswordAgeDays = 30, PasswordLength = 20, PasswordComplexity = 4; GPO: Computer Configuration > Administrative Templates > System > LAPS

**How to Verify:** Get-LapsADPassword -Identity prod-fs-02 -AsPlainText returns a password and an expiration time; Get-WinEvent -LogName Microsoft-Windows-LAPS/Operational shows password updates succeeding; Check that a second server has a different password

**Security Impact:** I

**Operational Impact:** I

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

**Security Impact:** I

**Operational Impact:** I

---

**Control Number:** 11

**Control Name:** Enable Advanced Audit Policy subcategories

**Default State:** I

**Hardened State:** I

**Implementation Method:** I

**Registry/GPO Path:** HKLM:\SYSTEM\CurrentControlSet\Control\Lsa\SCENoApplyLegacyAuditPolicy = 1 (DWORD); GPO: Computer Configuration > Windows Settings > Security Settings > Advanced Audit Policy Configuration > Audit Policies > Logon/Logoff

**How to Verify:** auditpol /get /category:"Logon/Logoff" shows the expected settings; Sign in and out, then check that events 4624, 4625, 4634, and 4672 appear in the Security log

**Security Impact:** I

**Operational Impact:** I

---

**Control Number:** 12

**Control Name:** Enable PowerShell Script Block Logging

**Default State:** 	Disabled: EnableScriptBlockLogging = 0

**Hardened State:** EnableScriptBlockLogging = 1

**Implementation Method:** I

**Registry/GPO Path:** HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging\EnableScriptBlockLogging = 1 (DWORD); GPO: Computer Configuration > Administrative Templates > Windows Components > Windows PowerShell > Turn on PowerShell Script Block Logging

**How to Verify:** Run a test command, then Get-WinEvent -FilterHashtable @{LogName='Microsoft-Windows-PowerShell/Operational'; Id=4104} shows the script text

**Security Impact:** I

**Operational Impact:** I

---

**Control Number:** 13

**Control Name:** Command-line process auditing

**Default State:** ProcessCreationIncludeCmdLine_Enabled = 0

**Hardened State:** ProcessCreationIncludeCmdLine_Enabled = 1

**Implementation Method:** I

**Registry/GPO Path:** HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System\Audit\ProcessCreationIncludeCmdLine_Enabled = 1 (DWORD); GPO: Computer Configuration > Administrative Templates > System > Audit Process Creation > Include command line in process creation events Advanced Audit Policy > Detailed Tracking > Audit Process Creation

**How to Verify:** 	Run whoami, then find event 4688 in the Security log and check that "Process Command Line" is filled in auditpol /get /subcategory:"Process Creation" shows Success

**Security Impact:** I

**Operational Impact:** I

---

**Control Number:** 15

**Control Name:** Object access auditing on sensitive shares

**Default State:** I

**Hardened State:** I

**Implementation Method:** Turn on Object Access > Audit File System with a GPO, then set the SACL:
$acl = Get-Acl C:\FinanceShare
$rule = New-Object System.Security.AccessControl.FileSystemAuditRule('Everyone','Modify,Delete,ChangePermissions,TakeOwnership','ContainerInherit,ObjectInherit','None','Success,Failure')
$acl.AddAuditRule($rule); Set-Acl C:\FinanceShare $acl

**Registry/GPO Path:** GPO: Computer Configuration > Windows Settings > Security Settings > Advanced Audit Policy Configuration > Audit Policies > Object Access > Audit File System
There is no registry key for this. The SACL is stored on the NTFS folder itself.

**How to Verify:** (Get-Acl C:\FinanceShare -Audit).Audit lists the rule; Create, edit, and delete a test file, then look for events 4663 (access attempt), 4660 (delete), and 4670 (permissions changed) in the Security log

**Security Impact:** I

**Operational Impact:** I

---

**Control Number:** 16

**Control Name:** AppLocker

**Default State:** I

**Hardened State:** I

**Implementation Method:** I

**Registry/GPO Path:** GPO: Computer Configuration > Windows Settings > Security Settings > Application Control Policies > AppLocker; Rules are stored under HKLM:\SOFTWARE\Policies\Microsoft\Windows\SrpV2

**How to Verify:** (Get-Service AppIDSvc).Status returns Running Get-AppLockerPolicy -Effective -Xml shows the rules; Use Test-AppLockerPolicy on a sample file; Events 8003 (audit) and 8004 (blocked) in Microsoft-Windows-AppLocker/EXE and DLL

**Security Impact:** I

**Operational Impact:** I

---

**Control Number:** 18

**Control Name:** PowerShell language mode

**Default State:** FullLanguage

**Hardened State:** ConstrainedLanguage

**Implementation Method:** Enforce AppLocker Script rules (or use WDAC). The __PSLockdownPolicy environment variable is for testing only, because it is not a real security boundary.

**Registry/GPO Path:** GPO: AppLocker > Script Rules set to Enforce; Rules are stored under HKLM:\SOFTWARE\Policies\Microsoft\Windows\SrpV2\Script

**How to Verify:** $ExecutionContext.SessionState.LanguageMode returns ConstrainedLanguage in a normal session; Check that an approved admin script still runs and that a test use of Add-Type is blocked

**Security Impact:** I

**Operational Impact:** I

---

**Control Number:** 19

**Control Name:** Attack Surface Reduction rules

**Default State:** I

**Hardened State:** I

**Implementation Method:** Add-MpPreference -AttackSurfaceReductionRules_Ids <GUID> -AttackSurfaceReductionRules_Actions AuditMode, then change to Enabled after the audit period. Defender Antivirus must be the main antivirus with real-time protection on.

**Registry/GPO Path:** HKLM:\SOFTWARE\Policies\Microsoft\Windows Defender\Windows Defender Exploit Guard\ASR\Rules; GPO: Computer Configuration > Administrative Templates > Windows Components > Microsoft Defender Antivirus > Microsoft Defender Exploit Guard > Attack Surface Reduction > Configure Attack Surface Reduction rules

**How to Verify:** (Get-MpPreference).AttackSurfaceReductionRules_Ids and .AttackSurfaceReductionRules_Actions list the rules and their modes; Events 1122 (audit) and 1121 (block) in Microsoft-Windows-Windows Defender/Operational

**Security Impact:** I

**Operational Impact:** I

---

**Control Number:** 20

**Control Name:** Require Network Level Authentication for RDP

**Default State:** UserAuthentication = 0

**Hardened State:** UserAuthentication = 1

**Implementation Method:** I

**Registry/GPO Path:** HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp\UserAuthentication = 1 (DWORD); GPO: Computer Configuration > Administrative Templates > Windows Components > Remote Desktop Services > Remote Desktop Session Host > Security > Require user authentication for remote connections by using Network Level Authentication

**How to Verify:** (Get-CimInstance -Namespace root\cimv2\TerminalServices -ClassName Win32_TSGeneralSetting).UserAuthenticationRequired returns 1; Try to connect from a client that does not support NLA and confirm it is refused

**Security Impact:** I

**Operational Impact:** I
