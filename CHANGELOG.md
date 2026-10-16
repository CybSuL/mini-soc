# CHANGELOG

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
