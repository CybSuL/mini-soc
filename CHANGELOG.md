\# CHANGELOG

&#x20;

\## 2026-10-12 | W2 Session 1/3: Windows Security events 4624, 4625, 4688

Did

\- Generated and read 4624 (lock and unlock) and 4625 (wrong password) in Event Viewer on my host PC.

\- Enabled Process Creation auditing and command-line logging, then generated 4688 events with notepad, whoami and dir.

\- Traced a parent-child process chain by matching Creator Process ID to New Process ID.

Learned

\- 4624 = successful logon, 4625 = failed logon, 4688 = new process started.

\- Logon Types: 2 = at the console, 10 = RDP.

\- Subject = who reported the logon. New Logon = who actually logged on. Names ending in $ are computer accounts.

\- 4625 Sub Status 0xC000006A = wrong password for a real account. 0xC0000064 = account does not exist.

\- 4688 and its command line are both off by default on Windows.

\- One program start = one 4688. Built-in cmd commands like dir create none.

\- Parent process matters: winword.exe launching cmd.exe usually means a malicious macro.


\## 2026-10-11 | W1 Session 2/2: Create the mini-soc repo

Did

\- Installed Git, set name and private GitHub email.

\- Created mini-soc on GitHub, cloned it, added TODO, CHANGELOG and README, pushed.

Learned

\- add = ready to commit, commit = local history, push = on GitHub.

\- origin = the name Git gives the GitHub cloned repo. origin/main = the main branch on GitHub.


\## 2026-10-10 | W1 Session 1/2: Git and GitHub basics

Did

\- Finished GitHub Skills "Introduction to GitHub": branch, commit, pull request, merge.

Learned

\- Commit = save point. Only committed work can be restored.

\- Branch = side path. Main stays untouched until merge.

\- Pull request asks to merge, merge accepts.

\- mini-soc uses main only. Commit whenever something works.

