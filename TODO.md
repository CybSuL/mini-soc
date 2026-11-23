# TODO

## W8: Detection #1-2: PowerShell abuse, local account creation (T1136). Learn the Sigma format.
- [x] W8 Session 1/5: Read a SigmaHQ encoded PowerShell rule, found it misses my W7 attack, installed sigma-cli, converted the rule to SPL (2026-11-22)
- [x] W8 Session 2/5: Wrote my own Sigma rule for encoded PowerShell flags, sigma check passes (2026-11-23)
- [ ] W8 Session 3/5: Convert my encoded PowerShell rule to SPL, test it in Splunk against the W7 attack in attack-log.csv, check for false positives, save the SPL in the detection folder
- [ ] W8 Session 4/5: Run Atomic test T1136.001 (local account creation) on WS01, log it in attack-log.csv, find Event ID 4720 in Splunk
- [ ] W8 Session 5/5: Write a Sigma rule for account creation, convert it to SPL, test it against attack-log.csv, document both detection folders

## W8 downloads
- [ ] T1136.001.yaml from the atomic-red-team repo, before W8 Session 4/5

## Before 2026-12-19
- [ ] Switch Splunk from the Enterprise Trial license to the Free license

## Done (details per session in CHANGELOG.md)
- [x] W1 (2026-10-10 to 2026-10-11): Git and GitHub basics, mini-soc repo created and pushed
- [x] W2 (2026-10-12 to 2026-10-14): Windows Security events 4624, 4625, 4688, Sysmon event IDs, MITRE ATT&CK basics
- [x] W3 (2026-10-15 to 2026-10-17): VMware, isolated network VMnet2, Ubuntu Server VM at 192.168.50.20 with SSH, basic Linux commands
- [x] W4 (2026-10-18 to 2026-10-22): Windows Server VM DC01 at 192.168.50.10, Splunk Enterprise on the Ubuntu VM, web UI and port 9997 working. Review video recorded 2026-10-22
- [x] W5 (2026-10-24 to 2026-10-28): DC01 promoted (forest minisoc.lab), OUs, users and groups, Windows 11 client WS01 at 192.168.50.100 joined. Review video recorded before 2026-11-02
- [x] W6 (2026-11-03 to 2026-11-15): Sysmon and Universal Forwarder on DC01 and WS01, Sysmon Event ID 1 in Splunk, data-flow diagram. Review video recorded 2026-11-15
- [x] W7 (2026-11-15 to 2026-11-22): Ubuntu VM isolated, attack-log.csv, Kali VM at 192.168.50.250, Atomic Red Team on WS01, T1059.001-17 run and found in Splunk. Review video skipped (videos now optional)
