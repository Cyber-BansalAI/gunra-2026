# Hunt guide: did Gunra get in, and did it touch our backups?

As of **2026-11-16**. Read [DISCLAIMER.md](../DISCLAIMER.md) first. We have not tested these commands on your systems.
Run them yourself, on a copy of the logs if you can, and adjust paths to your setup. Nothing in this package runs
anything for you, and every command shown only reads.

**Two questions, kept apart.**

1. *Is there sign the group was in our network?* Answered from firewall, VPN and sign-in records, and from files and
   tools on computers and servers.
2. *Did it go after our backups and our data?* Answered from backup storage, backup software, and records of large
   uploads.

The advisory describes the group stealing data first, then locking files, then threatening to post the stolen data on
a leak site if the victim does not pay. So a locked file is the **last** step. The signs below can appear before that.
The advisory does **not** say how long the group stays inside before it locks files, so do not assume a clean week
means nothing happened.

## 0. Set your window

- The agencies saw these indicators between **April 2025 and July 2026**. Search at least that far back if your
  records go that far.
- If your logs begin later, say so in your notes: you cannot rule anything out for the earlier period.

## 1. Read the cautions first

- The advisory says the everyday tools it lists are legitimate, and that using one is not proof of harm without other
  evidence. A match on a tool such as AnyDesk, WinRAR or FileZilla is a question for whoever installed it, not a
  finding.
- The agencies also say the addresses may be historical and should be vetted before acting. Look each one up before you
  block it.
- A clean result is not proof of safety. File names and hashes are easy to change.

## 2. The firewall and VPN first

The advisory says the group mostly got in by using known flaws in firewalls and VPN devices on the internet, and in one
case by logging in with a default administrator password on a VPN device that had no limit on wrong guesses. So start
with the device at the edge of your network, not with the laptops.

- **Is the device up to date?** The advisory names two flaws in certain FortiOS and FortiProxy versions (`GU` rows of
  type `cve` in `iocs/iocs.csv`). Ask your IT provider or the firewall's vendor notice whether your version is
  affected and when it was fixed.
- **Look at its administrator accounts.** The advisory says the two flaws let attackers add a permanent administrator
  account called `forticloud-sync`. Any administrator account nobody can explain is worth a call to your IT provider.
- **Look for default or unused accounts.** The advisory describes the group using a default administrator account, and
  editing an unused VPN account so it no longer had to change its password.
- **Look at VPN sign-ins at odd hours.** The advisory says the group mostly worked between 10:00 p.m. and 6:00 a.m.

Read the addresses from the dataset and search an exported firewall or proxy log for them (adjust the file names):

```
$ips = (Import-Csv .\iocs\iocs.csv | Where-Object { $_.type -eq 'ip' }).value
Select-String -Path .\firewall-export.log -SimpleMatch -Pattern $ips
```

How to read a match:

- Any match to an address in the dataset deserves a look at **when** and **what it was**. The agencies saw most of these
  addresses in 2025, so an old match matters more than a new one.
- A VPN or remote access sign-in from a place or provider your staff never use is worth asking about even with no
  match here.
- The dataset also lists one clearnet domain and three Tor addresses. Do not open them. If your DNS log is exportable,
  search it for the clearnet domain in the same way.

## 3. Files and tools on computers and servers

Search for the file names in the dataset. This lists matches without changing anything:

```
Get-ChildItem -Path C:\ -Recurse -Force -ErrorAction SilentlyContinue -Include cryptor.exe,msmp.exe,R3ADM3.txt,R34DM3.txt,'*.ENCRT' | Select-Object FullName, Length, LastWriteTime
```

If a file turns up, work out its hash and compare it with the `sha256` rows. `main.exe` is a generic name, so check
the hash and not the name:

```
Get-FileHash -Algorithm SHA256 -Path "C:\path\to\the\file"
```

How to read it:

- Files ending **.ENCRT** mean data was already locked on that computer. On Linux the advisory names **.GNRA**. Treat
  either as an incident (section 9).
- A ransom note named `R3ADM3.txt` (the advisory also spells it `R34DM3.txt`) in a folder means the same.
- The advisory lists common tools the group used: FileZilla, RClone, 7-Zip, WinRAR, AnyDesk, Google Remote Desktop,
  MobaXterm, DBeaver, Visual Studio Code, Slack, Mimikatz, Impacket, Sliver and Amass. We did not list their names as
  file-name indicators. Ask whether anyone at your company installed them.

## 4. Accounts and sign-ins

The advisory says the group stole password data from domain controllers and then moved between systems with those
passwords, and by remote desktop. On a Windows server or domain controller these read-only commands show the last 30 days
of new accounts and of people added to administrator groups:

```
Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4720; StartTime=(Get-Date).AddDays(-30)} | Select-Object TimeCreated, Message
Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4728,4732,4756; StartTime=(Get-Date).AddDays(-30)} | Select-Object TimeCreated, Message
```

Remote Desktop sign-ins (logon type 10) from addresses you do not recognise, especially at night, are also worth a
look:

```
Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4624; StartTime=(Get-Date).AddDays(-30)} | Where-Object { $_.Message -match 'Logon Type:\s+10' } | Select-Object TimeCreated, Message
```

These records need Windows auditing turned on. If they are empty, that is itself a finding: you may not be recording
what you need.

## 5. Backups

The advisory says the group deletes volume shadow copies before encryption, and at one victim deleted backup and
archived data at both the main data center and the disaster-recovery site, before and after the lock-up.

- **Backup storage and software.** Check its own admin log or ask your provider whether backups were deleted, had their
  settings changed or had their schedules changed by someone nobody remembers.
- **Both locations.** If your backups are in two places, check both. The advisory describes damage at both.
- **Volume Shadow Copy.** This command lists the service stopping or starting in the last 30 days. The service also
  stops by itself when idle, so a stop on its own proves little. Look for it alongside other signs:

```
Get-WinEvent -FilterHashtable @{LogName='System'; Id=7036; StartTime=(Get-Date).AddDays(-30)} | Where-Object { $_.Message -like '*Volume Shadow Copy*' } | Select-Object TimeCreated, Message
```

- **The test that matters.** Do you have a backup copy that is **offline or cannot be changed**, and has someone
  restored from it recently? The agencies recommend backups that are offline or immutable, stored somewhere separate,
  and tested.

## 6. Tunnels and remote tools you did not install

The advisory says the group downloaded an SSH tunnelling tool onto a compromised system from an outside server to keep
connections open between systems, and used remote-access software such as AnyDesk. Many companies use these tools
legitimately. List what is installed and ask who put it there:

```
Get-ChildItem "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall" | Get-ItemProperty | Where-Object { $_.DisplayName -match 'AnyDesk|OpenSSH|MobaXterm|Chrome Remote Desktop' } | Select-Object DisplayName, InstallDate
Get-Service -Name sshd -ErrorAction SilentlyContinue | Select-Object Name, Status, StartType
```

An entry nobody can explain, or an SSH service on a server that never needed one, is a reason to call your IT provider.

## 7. Data leaving

The advisory says the group copied business documents, databases, personal data and internal email before locking
files. It names a tool, called `main.exe`, that pulled files from OneDrive and SharePoint, and says it sent archives to
the Mega file-sharing service (up to tens of terabytes at one victim) and used FileZilla over FTP. Ask your IT provider
or firewall vendor for a report of the **largest outbound transfers** from servers and file shares over the last 90
days, and for large downloads from OneDrive and SharePoint by an account or application nobody recognises. Look for
uploads to storage services nobody at the company uses. Ordinary backups will also appear, so compare against what you
expect.

## 8. What the results mean

| You found | It suggests | Next step |
|---|---|---|
| A published address in the firewall or VPN log, nothing else | A connection to a reassigned address, or early activity | Look up the address, check the date and the account. Keep the logs |
| A tool such as AnyDesk, WinRAR or FileZilla nobody can explain | A possible foothold, or a tool someone forgot to mention | Ask who installed it. If no one did, treat it as an incident |
| A listed file name, or a hash that matches | Likely attacker tooling | Treat as an incident |
| An administrator account on the firewall nobody can explain, new administrator accounts, backups deleted, large uploads nobody expected | Likely an intruder preparing to lock files | Treat as an incident now: the advisory puts data theft and backup damage before the lock-up |
| Files ending .ENCRT or .GNRA, or a ransom note | Data is already locked | Section 9 |
| Nothing | Not proof of safety: these indicators change and your records may not capture what matters | Fix the basics in section 10 anyway |

## 9. If you think it is real

Preserve first, clean later:

1. Isolate the affected computers from the network, but do not turn them off if you can avoid it, and keep the logs
   unchanged.
2. Call your IT provider. Do not use the computers that may be affected to talk about it.
3. Report to the FBI's Internet Crime Complaint Center, a local FBI or Secret Service field office, or CISA. The agencies
   ask for logs showing connections to outside addresses, a sample ransom note, any messages from the group and a
   harmless sample of an encrypted file. The advisory lists the contact details.
4. If a **Linux** server was locked, keep the encrypted files, their timestamps, the ransom notes and the logs. The
   advisory says researchers found a weakness in that version of the program that may let a specialist rebuild the keys
   from file timestamps. Do not try this yourself.
5. The agencies do not encourage paying a ransom: payment is no promise that files will come back, and it can
   encourage further attacks.

## 10. Reduce the risk

These are drawn from the advisory's mitigations. Read the advisory for the full list.

- Patch the firewall and VPN first, and anything the CISA Known Exploited Vulnerabilities list names.
- Require MFA on email, VPN and every account that can reach important systems.
- Remove or lock default and unused accounts on the VPN and firewall.
- Keep backups that are offline or cannot be changed, in a separate place, and test a restore.
- Limit administrator accounts, review them regularly, and watch for new ones.
- Split the network into parts so one compromised computer cannot reach everything.

## Not sure what you found?

Bring the log lines to your IT person first. If you want a second pair of eyes,
[SOC-Lite by BansalAI](https://cyber.bansalai.com/?ref=github) offers a free 20-minute
[visibility check](https://cyber.bansalai.com/?ref=github#book).

## Source for this guide

The joint advisory AA26-222A by the FBI, CISA, the Department of Defense Cyber Crime Center, the National Security
Agency, the U.S. Secret Service and South Korea's National Police Agency (full credit and link in
[CREDITS.md](../CREDITS.md)). Windows event numbers are standard Windows records and are not from the advisory.
