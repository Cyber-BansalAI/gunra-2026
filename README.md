# Gunra ransomware (CISA advisory AA26-222A): indicators and hunt guide

> **Read first.** This package is aggregated from a public government advisory by SOC-Lite by BansalAI. It is not tested
> by us, it comes with no warranty, and it is not professional advice. It is as of **2026-11-16** and will go stale. See
> [DISCLAIMER.md](DISCLAIMER.md). We are not affiliated with any agency credited in [CREDITS.md](CREDITS.md), and none
> of them reviewed or endorsed this package.

On 2026-08-10 the FBI, CISA, the Department of Defense Cyber Crime Center, the National Security Agency, the U.S. Secret
Service and South Korea's National Police Agency published a joint advisory on **Gunra**, a ransomware group the FBI
first saw in April 2025. The advisory says Gunra started as a lock-and-steal ransomware built on leaked source code, and
in early 2026 began renting itself out to other criminals as a "ransomware as a service". It describes how the group
gets in, how it moves through a network, how it steals data and attacks backups, and lists the indicators the agencies
found.

This repository copies **those indicators** (with credit) and adds a short guide to checking your own systems for them.

**Not technical? Start with the plain-language version:**
[Gunra Ransomware Gets In Through the Firewall: What to Ask Your IT Person](https://blog.bansalai.com/news/gunra-ransomware-firewall-vpn-backups/?ref=github)
on the SOC-Lite blog. It gives four questions to put to your IT person or provider.

## What is in here

| File | What |
|---|---|
| [`iocs/iocs.csv`](iocs/iocs.csv) | One row per indicator, with the source, the date, how fragile it is, and a review date |
| [`docs/hunt-guide.md`](docs/hunt-guide.md) | What to check on your firewall and VPN, on Windows computers and servers, and on backup storage, what the results do and do not mean, and what to do if something looks real |
| [`CREDITS.md`](CREDITS.md) | The agencies whose advisory this is built from, and what we took from it |
| [`DISCLAIMER.md`](DISCLAIMER.md) | The full disclaimer |

## Where the indicators come from

**Rows:** 50 · **verified (two or more sources):** 0 · **single source:** 50

These counts match [`iocs/iocs.csv`](iocs/iocs.csv); we check them every time we change the file
(a test checks them in our private working copy). Every value was read on the advisory's own page, and checked against
the agencies' machine-readable copy. One source means no row is verified yet. What we took, and where it came from, is
in [CREDITS.md](CREDITS.md).

### Copied into `iocs.csv` (indicator values, with credit)

| Source | Rows | Rows by type | What we took | Link |
|---|---|---|---|---|
| **CISA and partner agencies (advisory AA26-222A)** | 50 | 17 ip, 4 domain, 4 sha256, 1 sha1, 1 md5, 6 filename, 3 string, 1 account, 2 cve, 1 command, 10 behaviour | The indicator tables: addresses, domains, file hashes, file names, an account name and chat IDs, and how the group works | https://www.cisa.gov/news-events/cybersecurity-advisories/aa26-222a |
| **Total** | 50 | | | |

### Linked, not copied

| Source | What it has | Why we link | Link |
|---|---|---|---|
| **The advisory itself** | Machine-readable STIX files of the indicators, four ransom-negotiation email addresses, a mapping of the group's methods, and mitigations | It is the agencies' own, current copy. If you load indicators into a security tool, take them from there | https://www.cisa.gov/news-events/cybersecurity-advisories/aa26-222a |

Everyday tools the advisory lists (such as FileZilla, RClone, 7-Zip, WinRAR, AnyDesk and the Impacket toolkit) are
legitimate and common, so their names are not listed as file-name rows. The hunt guide describes them in words, and a
few behaviour rows describe how the group used them.

## Read the results carefully

- **A match is a reason to look, not a verdict.** The advisory says the tools it lists are legitimate, that using one is
  not proof of harm on its own, and that addresses should be investigated or vetted before acting, such as blocking.
- **Do not mass-block addresses.** The agencies saw most of these between mid-2025 and early 2026 and warn that they may
  be historical. An address used for harm once can later belong to a harmless service. Look each one up against your own
  logs first.
- **We left out everyday tools' names as file-name rows on purpose.** Listing them would bury a real result in false
  alarms. `main.exe` is a generic name, so we list its two hashes and not the name by itself.
- **The advisory is one source.** Every row comes from it, so no row is marked verified. We mark a row verified only
  when two or more independent sources report it.
- **These indicators go stale quickly.** The agencies observed them between April 2025 and July 2026. File names and
  hashes are cheap for an attacker to change, so a clean result is not an all-clear.
- **The advisory does not say how long the group stays inside a network before it locks files.** We do not guess. The
  hunt guide treats new accounts, tunnels and backup changes as warning signs on their own.
- **There is more in the advisory than we copy.** It maps the group's methods and lists mitigations and response steps.
  Read the advisory itself for them (link in [CREDITS.md](CREDITS.md)).

## Using `iocs.csv`

| Column | Meaning |
|---|---|
| `id` | Stable row ID (`GU-nnnn`) |
| `wave` | Always `gunra` in this release |
| `type` | `ip`, `domain`, `sha256`, `sha1`, `md5`, `filename`, `string` (a chat ID), `account`, `cve`, `command` or `behaviour` |
| `value` | The indicator. Addresses and domains are written as normal text, not defanged. Never open them in a browser |
| `match` | `exact`, `substring` or `wildcard` |
| `evidence` | Where it would show: `network`, `file`, `process` or `event-log` |
| `fragile` | `yes` if the attacker can change it trivially |
| `verified` | `yes` only when two or more independent primary sources report it. `no` means a single source |
| `source_url`, `source_date` | Where we took it from and the date of that source |
| `fetched`, `review_by` | When we read it, and when it should be re-checked. Addresses and domains have short review dates |
| `notes` | Context and caveats. Sentences starting "Our note" are ours, not the agencies' |

Search your own systems with the steps in the hunt guide. This package does not ship anything that runs on your
systems or collects data from them.

## Licence

The text we wrote is licensed under CC BY 4.0 ([LICENSE](LICENSE)). The indicator values and facts belong to the
agencies that reported them: they are credited in [CREDITS.md](CREDITS.md), and we do not relicense them. Details and
how to credit us are in [NOTICE.md](NOTICE.md).

## Where to go for the full, current picture

The advisory is the source for everything here and may be updated. For its complete tables, its mapping of the group's
methods and its mitigations, follow the link in [CREDITS.md](CREDITS.md). Near the top of that page the agencies also
offer the indicators as machine-readable STIX files (XML and JSON). If you load indicators into a security tool, take
them from there, because that is the agencies' own copy.

## Help with a suspected compromise

If something looks real, preserve the logs first, isolate the affected computers, and call your IT provider. The agencies
also ask victims to report to the FBI (see the advisory). [SOC-Lite by BansalAI](https://cyber.bansalai.com/?ref=github)
helps small and mid-sized Central Florida businesses review alerts and work with their IT person or MSP.
[Book a free 20-minute visibility check](https://cyber.bansalai.com/?ref=github#book). More plain-language security
write-ups are on the [SOC-Lite blog](https://blog.bansalai.com/?ref=github).
