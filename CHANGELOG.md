# CHANGELOG

## 2026-11-26 | W8 Session 4/6: Prepare the local account creation attack (T1136.001)
Did
- Split the old W8 Session 4 into two short sessions: prepare (today) and run (next). W8 now has 6 sessions.
- Downloaded only T1136.001.yaml from the atomic-red-team repo to E:\Mini-SOC Project\AtomicStaging\atomics\T1136.001.
- Copied it to C:\AtomicRedTeam\atomics on WS01, signed in as sul.admin.
- Listed the Windows tests with -ShowDetailsBrief: 4 (net user in cmd), 5 (New-LocalUser in PowerShell), 8 (new admin user), 9 (via .NET). Ran nothing.

Learned
- T1136.001 = Create Account: Local Account. Attackers create a local account as a backdoor to get back in later (Persistence).
- One technique, many procedures: the 4 tests create an account in 4 different ways.
- A rule matches text, not meaning. A rule on "net user" in CommandLine catches tests 4 and 8 but misses 5 (New-LocalUser) and 9 (.NET, no net.exe process).
- Event ID 4720 (a user account was created) is written whatever tool is used. Detect on the event, not the command.

## 2026-11-24 | W8 Session 3/5: Test my encoded PowerShell rule in Splunk
Did
- Converted my rule to SPL with sigma-cli (sysmon and splunk_windows pipelines).
- Ran it in Splunk with index=windows over the last 30 days: 1 event, the W7 attack T1059.001-17 on WS01 (2026-11-20 05:40 UTC, parent cmd.exe).
- Scored against attack-log.csv: 1 true positive, 0 false positives, 0 missed attacks. The SigmaHQ rule would have missed it.
- Saved the ready-to-run SPL (with index=windows) as detections/T1059.001-encoded-powershell/splunk.spl.

Learned
- windash turns each value into 5 variants: - / – — ―. PowerShell accepts all of them as a parameter prefix, and attackers use the look-alike dashes to evade rules that only search for -. 4 flags became 20 CommandLine values.
- In SPL, a space between conditions means AND.
- | table field1 field2 shows only chosen fields as columns, easier than reading raw events.
- Splunk shows times in my time zone: 05:40 UTC appears as 08:40 Riyadh time.
- sigma convert -o writes the output file directly, which keeps special characters intact.
- 0 false positives in my lab does not mean 0 in a real company. Tools like SCCM and Intune would trigger it, so level stays medium.
- attack-log.csv shows both what fired correctly and what should have fired but did not.

## 2026-11-23 | W8 Session 2/5: Write my own Sigma rule for encoded PowerShell
Did
- Split W8 into 5 shorter sessions because of the holiday week. Writing the rule (Session 2) and testing it (Session 3) are now separate.
- Created detections/T1059.001-encoded-powershell/proc_creation_win_powershell_encoded_flag.yml.
- The rule matches powershell.exe or pwsh.exe AND an encoded flag (' -e ', ' -en ', ' -ec ', ' -enc') with the windash modifier. Level medium.
- Replaced an invalid id with a fresh GUID. sigma check passes with 0 errors and 0 issues.

Learned
- PowerShell accepts any unambiguous short form of a parameter: -e, -en, -enc up to -EncodedCommand, plus the alias -ec.
- ' -e ' with a trailing space matches only that flag, not -ExecutionPolicy. ' -enc' needs no space because every longer form starts with it.
- The windash modifier makes - also match /.
- YAML: values starting with a space need quotes. Indent with spaces, never tabs.
- condition with or = either selection is enough, so every PowerShell start would alert. and = both must match.
- Encoded command = a script passed as Base64 so quotes, $ and newlines survive. Legit users: SCCM, Intune, monitoring agents, installers. Attackers use it to hide the content. Same flag, different intent, so level medium.
- My rule matches the flag, not the Base64 content, so it catches my W7 attack (JgA) that the SigmaHQ rule missed.
- Every Sigma rule needs a unique UUID as its id (New-Guid). sigma check also checks SigmaHQ style, like a filename length of 10 to 90 characters.

## 2026-11-22 | W8 Session 1/4: Learn the Sigma format
Did
- Made weekly review videos optional and skipped the W7 video.
- Read the SigmaHQ rule "Suspicious Encoded PowerShell Command Line" (proc_creation_win_powershell_base64_encoded_cmd.yml).
- Checked it by hand against my W7 attack: my Base64 starts with JgA (encoded "&"), which is not in the rule's prefix list, so the rule would not fire.
- Installed sigma-cli 4.0.1, pysigma-backend-splunk 3.0.1 and the sysmon pipeline plugin on my host.
- Downloaded the rule to E:\Mini-SOC Project\sigma-practice (outside the repo) and converted it to SPL.
- Skipped running the converted SPL in Splunk. Testing happens in Session 2 with my own rule.

Learned
- Sigma = a detection format written once and converted to SPL, KQL and others.
- Rule parts: metadata (title, id, status, description, references, author, date), tags, logsource, detection, falsepositives, level.
- logsource is generic: category process_creation covers both Sysmon Event ID 1 and Security 4688.
- detection = named selections plus a condition that combines them. Several values under one field = OR. "all of selection_cli_*" = every selection starting with selection_cli_ must match.
- Modifiers in SPL: endswith becomes "*\\x", contains becomes "*x*".
- The first Base64 characters after powershell -e reveal how the command starts: JAB = "$", JgA = "&".
- This rule needs " -e" AND a known Base64 prefix. Starting the command with "&" evades it.
- " -e" alone also matches -ExecutionPolicy and -ErrorAction. Too broad = alert flood, too narrow = missed attacks.
- level (informational, low, medium, high, critical) = how urgently an analyst should look. It mixes impact and confidence.
- falsepositives = known harmless triggers. This rule has none, which is a gap.
- Tags link to ATT&CK: attack.execution = tactic, attack.t1059.001 = technique.
- sigma-cli: -t = target language, -p = pipelines that translate generic names to my log names (lower priority number runs first). index=windows must be added by hand.

## 2026-11-20 | W7 Session 3/3: Run the first attack and find it in Splunk
Did
- Read Atomic test T1059.001-17 (PowerShell Command Execution) with -ShowDetails before running it: cmd.exe executor, no admin, no dependencies.
- Ran it on WS01 as MINISOC\alice.smith at 05:40 UTC. It printed "Hello, from PowerShell!". No cleanup needed.
- Logged the run as the first row of attack-log.csv.
- Found it in Splunk: Sysmon Event ID 1, Image powershell.exe, CommandLine powershell.exe -e <Base64>, ParentImage cmd.exe, User MINISOC\alice.smith.
- Traced the chain: my PowerShell (parent explorer.exe) started cmd.exe, which started powershell.exe -e.
- Decoded the Base64 on my host: & (gcm ('ie{0}' -f 'x')) ("Wr"+"it"+"e-H"+"ost ..."), which is Invoke-Expression "Write-Host 'Hello, from PowerShell!'".

Learned
- Always read an Atomic test (-ShowDetails) before running it.
- SPL: Field=value filters a field. "image: PowerShell" is just two keywords and matches any event containing them.
- Image="*\\powershell.exe" excludes splunk-powershell.exe. In SPL, \\ is one literal backslash.
- Too many similar events: filter on fields, compare with | table, follow ParentProcessId to ProcessId, and explain away every event that is not the attack.
- splunk-powershell.exe with parent splunkd.exe is the forwarder's own noise.
- The User field shows NOT_TRANSLATED plus the real user. Trust the Sysmon value.
- powershell -e takes Base64 of UTF-16LE text. Decode with [System.Text.Encoding]::Unicode.GetString([Convert]::FromBase64String('...')). Decoding does not run it.
- Obfuscation splits words ("Wr"+"ite") and builds names at run time ('ie{0}' -f 'x' = iex) so filters looking for iex or Write-Host miss it.
- Obfuscation hides the content but leaves its own fingerprint: the -e flag and an unusual parent like cmd.exe.
- attack-log.csv is the answer key for testing a detection: fired at the logged time = true positive, silent = missed attack, fired at other times = false positive.

## 2026-11-19 | W7 Session 2/3: Get Atomic Red Team onto WS01
Did
- Downloaded the Invoke-AtomicRedTeam module and its dependency powershell-yaml with Save-Module (download only, no install) to E:\Mini-SOC Project\AtomicStaging, outside the repo.
- Downloaded only the T1059.001 test definition (T1059.001.yaml) from the atomic-red-team GitHub repo. No Defender exclusion on my host.
- Copied Modules and atomics to C:\AtomicRedTeam on WS01 with VMware Tools copy-paste, signed in as sul.admin.
- Added a Defender exclusion on WS01 for C:\AtomicRedTeam only.
- Loaded the module and listed the T1059.001 tests with -ShowDetailsBrief: 21 tests for Windows. Ran nothing.
- Took snapshot atomic-ready on ws01. DC01 unchanged since pre-attacks-sysmon-uf.

Learned
- Invoke-AtomicRedTeam = the engine that runs tests. atomics\<technique>\<technique>.yaml = the tests for one technique. One technique has many tests (procedures).
- Downloading a file does not run it. Atomic tests are text; nothing executes until I run Invoke-AtomicTest.
- Attack tools stay outside the public repo, or git add . pushes them to GitHub.
- Set-ExecutionPolicy Bypass -Scope Process allows scripts in that window only.
- Adding a Defender exclusion is an attacker technique (T1562.001). It shows as Event ID 5007 in the Defender Operational log, which my forwarder does not collect yet.
- A path exclusion only protects files. Defender can still block suspicious PowerShell behavior at run time.
- Many tests need the internet (download cradles) and will fail in the isolated lab.
- Rolling back only WS01 is safe while its computer account password has not changed since the snapshot. "Trust relationship failed" means it has.

## 2026-11-17 | W7 Session 1/3: Lab prep and the Kali VM (started 2026-11-15)
Did
- Recorded the W6 review video.
- Disconnected the NAT adapter on the splunk VM. No internet, DC01 and the web UI still reachable.
- Boot then hung on systemd-networkd-wait-online. Added optional: true to ens34 (NAT) in netplan, applied with netplan try. Boot no longer hangs.
- Created attack-log.csv in the repo to record every attack test (date, UTC time, host, technique, test ID, notes).
- Took snapshot pre-attacks-sysmon-uf on dc01, ws01 and splunk.
- Downloaded the Kali VMware image, checked its SHA256 against kali.org, extracted it with 7-Zip.
- Opened it as the kali VM: 4 GB RAM, 2 cores, one adapter on VMnet2, set before the first boot.
- Set static IP 192.168.50.250/24, DNS 192.168.50.10, no gateway with nmcli. Ping to 8.8.8.8 fails.
- Changed the default kali password.
- Fixed the invisible mouse cursor by changing the VM's hardware compatibility from Workstation 8.x to 25H2. Toggling 3D acceleration did nothing; a software cursor worked but lagged.
- Turned off screen lock and sleep in Kali.
- Kali reaches Splunk (ping) and DC01's DNS (nslookup minisoc.lab).
- Took snapshot clean-static-ip on kali.

Learned
- At boot, Ubuntu waits for every netplan interface. One with dhcp4 and no cable waits until the 2-minute timeout. optional: true skips it.
- netplan try rolls a change back after 120 seconds unless confirmed, so a mistake cannot lock me out of SSH.
- Netplan files are root-only because they can hold secrets like Wi-Fi passwords.
- NO-CARRIER = the interface has no cable connected.
- Log attack times in UTC, like Splunk, so log rows and events match without converting.
- A matching hash proves the download is not corrupted or tampered with.
- An attacker VM goes on the isolated network before its first boot. The Kali image ships on NAT.
- VMnet2 has no DHCP, so every VM needs a static IP.
- Windows Firewall blocks inbound ping by default. Test DC01 another way, like DNS.
- Default credentials are an attack path (T1078.001). Change them even in a lab.
- Old virtual hardware can break a new guest kernel. When a VM image misbehaves, check its hardware compatibility first.
- An Xorg config that names a missing graphics driver stops the desktop from starting. Ctrl+Alt+F3 gives a text login to undo it.
- nslookup "Server: 192.168.50.10#53" = the DNS server's IP and the port it answered on (53 = DNS).

## 2026-11-14 | W6 Session 3/3: Find Sysmon Event ID 1 in Splunk and draw the data flow
Did
- Ran whoami /groups on WS01 and found its Sysmon Event ID 1 in Splunk with a search I wrote myself.
- Read Image, CommandLine, User and ParentImage in that event.
- Drew the lab data flow from memory, then added docs/architecture.svg to the README.
- Added the forwarder config to the repo as configs/splunk-uf/inputs.conf.

Learned
- SPL: field names are case-sensitive, field values are not. Quote values that contain anything other than letters and numbers.
- Sysmon Event ID 1 answers four questions: what ran (Image), how (CommandLine), who (User), from where (ParentImage).
- An unusual parent is a red flag: WINWORD.EXE starting whoami points to a malicious macro doing Discovery (T1033).
- Data flow: Sysmon records the process, writes it to the Sysmon Operational log, the forwarder reads it (inputs.conf) and sends it on port 9997, Splunk stores it in the index windows, I search on port 8000.
- WS01 needs DC01 for DNS and for domain logins.
- inputs.conf has no secrets, so it is safe to publish. Never commit etc\passwd or etc\auth.
- Splunk settings go in etc\system\local: upgrades overwrite default, local overrides it and survives.
- Copying a command on the host replaces whatever a VM put on the clipboard.

## 2026-11-13 | W6 Session 2/3: Install the Universal Forwarder on DC01 and WS01 (started 2026-11-12)
Did
- Found DC01 and WS01 clocks 10 hours ahead of real time. Turned on VMware Tools "Synchronize guest time with host" on both, set the time zone to Arab Standard Time on both and Asia/Riyadh on the Splunk VM.
- Created the index windows in Splunk.
- Downloaded Universal Forwarder 10.6.1.1 with winget download (no Splunk account needed).
- Installed the forwarder on WS01 with the wizard and on DC01 silently with msiexec. Both send to 192.168.50.20:9997.
- Wrote inputs.conf on both: Security and Sysmon Operational logs to the index windows.
- Sysmon events from WS01 were missing. splunkd.log showed errorCode=5. Switched the SplunkForwarder service to Local System on both.
- Verified with index=windows | stats count by host, source: Security and Sysmon arrive from DC01 and WS01.
- Skipped the snapshot. Last snapshots are still from W5.

Learned
- Compare clocks in UTC. Time zones can differ, the actual time cannot.
- WS01 was not syncing time from DC01: its source was Local CMOS Clock.
- VMware Tools time sync only moves a guest clock forward, never back.
- Index = separate storage in Splunk. Searches and detections target it by name.
- Port 8000 = web UI for people. Port 9997 = data coming in from forwarders.
- inputs.conf has one [WinEventLog://channel] block per log, with disabled = 0 and the target index.
- Write Splunk .conf files as ascii. A UTF-8 BOM can break the first block, and that log silently never arrives.
- Events missing: check the forwarder's own log (splunkd.log) before guessing. errorCode=5 = Access denied.
- Since 9.1, the forwarder runs as NT SERVICE\SplunkForwarder, which can read Security but not Sysmon. Local System fixes it in the lab. In production, grant read access to the Sysmon channel only (least privilege).
- A forwarder must not be newer than the Splunk server.
- "Last 15 minutes" is a moving window, so counts change between runs.

## 2026-11-03 | W6 Session 1/3: Install Sysmon on DC01 and WS01
Did
- Copied Sysmon and the SwiftOnSecurity config to C:\Tools\Sysmon on DC01 and WS01 with VMware Tools copy-paste.
- Installed Sysmon with the config on both. Service Sysmon64 is running on both.
- Saw Sysmon events in the Sysmon Operational log on WS01.
- Noticed that WS01's clock showed 7:29 PM while my host showed 9:30 AM. Moved the clock check to Session 2.

Learned
- .\ runs a file from the current folder only. "Not recognized" means the wrong folder: check with Get-Location. Extract All can create a subfolder.
- Installing a service needs admin rights. In a non-admin PowerShell, the Sysmon install fails and no service is created.
- Sysmon goes on the DC too: domain account creation, Kerberos ticket requests and lateral movement all leave evidence there.
- Sysmon Event ID 1 adds what 4688 lacks: hashes, ProcessGuid (PIDs get reused, GUIDs don't), parent command line. 4688 is not enabled on DC01 or WS01.
- All lab machines must agree on the real time. Time zones can differ because Splunk stores events in UTC. Kerberos fails above 5 minutes of difference, and a wrong clock hides events from searches like "Last 15 minutes".

## 2026-10-28 | W5 Session 3/3: Build the Windows client and join it to the domain
Did
- Created the ws01 VM (4 GB RAM, 2 vCPU, 60 GB disk, virtual TPM, VMnet2 only) and installed Windows 11 Enterprise Evaluation with a local account.
- Installed VMware Tools for copy and paste.
- Set static IP 192.168.50.100 and DNS 192.168.50.10.
- Joined minisoc.lab with sul.admin, renamed the PC to WS01, logged in as MINISOC\alice.smith.
- Moved WS01 from the Computers container to the Workstations OU.
- Took snapshots clean-install-static-ip (ws01, before the join), domain-joined (WS01) and ws01-joined (DC01).

Learned
- The client's DNS server is DC01. DC01 off = DNS timeouts, so no domain join and no domain logins.
- New computers land in the Computers container. Move them to an OU so Group Policy can apply.
- Normal users cannot change system settings (least privilege). A phished user gives the attacker only those rights.
- Snapshot the DC and clients together. Rolling back only one can break the computer account password they share (trust relationship failed).

## 2026-10-25 | W5 Session 2/3: Create OUs, users and groups
Did
- Created OUs Staff, Admins, Groups and Workstations under minisoc.lab.
- Created users alice.smith and bob.jones in Staff.
- Created my admin account sul.admin in Admins and added it to Domain Admins.
- Created the Global security group Finance in Groups, with Alice and Bob as members.
- Took snapshot ad-users-groups.

Learned
- OU vs container: both hold objects, but only OUs can have Group Policy applied and management delegated.
- Admins use a personal admin account, not the built-in Administrator: attackers know that name, and it hides who did what.
- Every new account is automatically in Domain Users.
- Permissions go to groups, not people. On a role change, remove the old group too, or the access stays (privilege creep).

## 2026-10-24 | W5 Session 1/3: Promote DC01 to a domain controller
Did
- Installed the AD DS role with its management tools.
- Promoted DC01 to the first domain controller of a new forest: minisoc.lab (short name MINISOC), with DNS installed.
- Verified with Get-ADDomain, nslookup and the domain controller SRV record lookup.
- Took snapshot dc-promoted.

Learned
- AD = Microsoft's directory: a database of users, computers and groups, plus services like checking logins.
- Domain = users and computers managed centrally under one set of rules.
- Forest = the top container, holding one or more domains. It is the security boundary.
- DC = the server that holds the AD database and checks logins.
- Clients find the DC through a DNS SRV record (_ldap._tcp.dc._msdcs.minisoc.lab). DNS only says where the DC is; the DC checks the password (Kerberos).

## 2026-10-21 | W4 Session 3/3: Reach the Splunk web UI and open the receiving port
Did
- Turned on the Ubuntu firewall (ufw) and allowed only the lab network on ports 22, 8000 and 9997.
- Logged in to the Splunk web UI from my host at http://192.168.50.20:8000.
- Turned on receiving on port 9997 and tested it from DC01 with Test-NetConnection.
- Took snapshot splunk-ready.

Learned
- Ports: 22 = SSH, 8000 = Splunk web UI, 9997 = logs from forwarders, 8089 = Splunk management.
- A connection needs both: a service listening on the port and a firewall rule allowing it. Check both when it fails.
- Add the SSH rule before enabling the firewall, or new SSH connections get blocked.
- The web UI uses http, so the password crosses the network unencrypted. Fine on an isolated lab network only.

## 2026-10-20 | W4 Session 2/3: Install Splunk Enterprise on the Ubuntu VM
Did
- Installed Splunk Enterprise 10.6.1.1, running as the splunk user.
- Set Splunk to start on boot and confirmed it after a reboot.
- Took snapshot splunk-installed.

Learned
- Least privilege: Splunk listens on network ports (8000, 9997), so it runs as the splunk user, not root.
- A setting is not proven until tested: reboot, then check the service.

## 2026-10-18 | W4 Session 1/3: Build the Windows Server VM
Did
- Created the dc01 VM (4 GB RAM, 2 vCPU, 60 GB disk, VMnet2 only) and installed Windows Server 2022 Standard Evaluation (Desktop Experience).
- Set static IP 192.168.50.10, mask 255.255.255.0, no gateway, DNS 192.168.50.10.
- Ping from DC01 to Ubuntu worked.
- Took snapshot clean-install-static-ip.

Learned
- DC = domain controller. It manages all users, computers and logins in the domain. Owning it means owning the domain.
- DC01 has no NAT adapter: it stays isolated, and a second DNS server from NAT would break AD.
- No gateway: VMnet2 has no router, and machines on the same subnet do not need one.
- A DC uses itself for DNS because domain computers find it through DNS.

## 2026-10-17 | W3 Session 3/3: Basic Linux commands
Did
- Read SSH logs with journalctl and filtered them with grep.
- Caused a failed SSH login on purpose and found it in the logs.
- Checked the SSH service with systemctl.

Learned
- Files starting with . are hidden from ls by default. Always use ls -la, attackers hide files this way.
- A disabled service does not start on boot. SSH still works because ssh.socket is enabled and starts it on the first connection.

## 2026-10-16 | W3 Session 2/3: Build the Ubuntu Server VM
Did
- Created the splunk VM.
- Installed Ubuntu Server 26.04.1 LTS.
- Installed OpenSSH server and connected from my host with ssh.
- Updated all packages with apt.
- Took snapshot clean-install-updated.

Learned
- Ubuntu Server has no desktop. Fewer resources used, less to attack, managed over SSH.

## 2026-10-15 | W3 Session 1/3: VMware and the isolated lab network
Did
- Installed VMware Workstation Pro. Downloaded the Ubuntu Server and Windows Server 2022 ISOs.
- Created VMnet2: host-only, DHCP off, subnet 192.168.50.0/24. Host adapter is 192.168.50.1.

Learned
- Bridged = VM joins my home network. NAT = VM gets internet through the host. Host-only = private network for VMs and host only, no internet.
- Host-only keeps attack traffic inside the lab.
- DHCP is off because other machines depend on fixed IPs: the forwarder sends to Splunk, clients use the DC for DNS.

## 2026-10-14 | W2 Session 3/3: MITRE ATT&CK basics
Did
- Explored the ATT&CK Enterprise matrix and read technique pages.
- Mapped this week's activity to techniques: T1110.001, T1059.001, T1547.001.

Learned
- Tactic = why (the goal). Technique = how (the method). Sub-technique = how exactly. Procedure = how a specific attacker did it.
- Mapping method: open the parent technique, compare only the sub-technique names, then confirm the breadcrumb says Enterprise and the first sentence of the description matches.
- Reconnaissance happens before getting in. Discovery happens after (e.g. whoami).
- One technique has many procedures. Detecting one tool can miss the same technique done another way.
- ID prefixes: T = technique, TA = tactic, S = software, G = group.
- Run key = Persistence. Dumping passwords from LSASS = Credential Access. Sending data out = Exfiltration.

## 2026-10-13 | W2 Session 2/3: Sysmon event IDs
Did
- Installed Sysmon on my host PC with the SwiftOnSecurity config.
- Generated and read Sysmon events 1, 3, 11 and 22 using whoami, Test-NetConnection and Invoke-WebRequest.
- Linked events from one process using ProcessGuid.

Learned
- Sysmon has its own log: Applications and Services Logs > Microsoft > Windows > Sysmon > Operational.
- Sysmon IDs: 1 = process created, 3 = network connection, 11 = file created, 13 = registry value set, 22 = DNS query. 4 and 16 = Sysmon stopped or config changed, possible tampering.
- A hash is the fingerprint of a file. Renaming the file does not change it.
- Event 22 + Event 3 show which program connected to which domain.
- ProcessGuid links all events from one run of a program.
- The config decides what gets logged. No log does not prove nothing happened.

## 2026-10-12 | W2 Session 1/3: Windows Security events 4624, 4625, 4688
Did
- Generated and read 4624 (lock and unlock) and 4625 (wrong password) in Event Viewer on my host PC.
- Enabled Process Creation auditing and command-line logging, then generated 4688 events with notepad, whoami and dir.
- Traced a parent-child process chain by matching Creator Process ID to New Process ID.

Learned
- 4624 = successful logon, 4625 = failed logon, 4688 = new process started.
- Logon Types: 2 = at the console, 10 = RDP.
- Subject = who reported the logon. New Logon = who actually logged on. Names ending in $ are computer accounts.
- 4625 Sub Status 0xC000006A = wrong password for a real account. 0xC0000064 = account does not exist.
- 4688 and its command line are both off by default on Windows.
- One program start = one 4688. Built-in cmd commands like dir create none.
- Parent process matters: winword.exe launching cmd.exe usually means a malicious macro.

## 2026-10-11 | W1 Session 2/2: Create the mini-soc repo
Did
- Installed Git, set name and private GitHub email.
- Created mini-soc on GitHub, cloned it, added TODO, CHANGELOG and README, pushed.

Learned
- add = ready to commit, commit = local history, push = on GitHub.
- origin = the name Git gives the GitHub cloned repo. origin/main = the main branch on GitHub.

## 2026-10-10 | W1 Session 1/2: Git and GitHub basics
Did
- Finished GitHub Skills "Introduction to GitHub": branch, commit, pull request, merge.

Learned
- Commit = save point. Only committed work can be restored.
- Branch = side path. Main stays untouched until merge.
- Pull request asks to merge, merge accepts.
- mini-soc uses main only. Commit whenever something works.



