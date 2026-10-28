# CHANGELOG

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
