# Credits and sources

This package is aggregated from one public advisory, written by the agencies below. **We thank them for publishing
it.** Each indicator in `iocs/iocs.csv` names the source it came from. Being listed here does not mean any of them
reviewed, tested or endorsed this package.

We copy facts (indicator values and figures) with attribution, and link to everything else. Where the advisory holds
more than we reproduce, follow the link for the full, current version.

## The advisory

| Who | What we used | Link |
|---|---|---|
| **#StopRansomware: Gunra Ransomware** (advisory AA26-222A, first published 2026-08-10), a joint advisory by the **Federal Bureau of Investigation (FBI)**, the **Cybersecurity and Infrastructure Security Agency (CISA)**, the **Department of Defense Cyber Crime Center (DC3)**, the **National Security Agency (NSA)**, the **U.S. Secret Service (USSS)** and the **Republic of Korea's National Police Agency (KNPA)** | The indicator tables (addresses, domains, file hashes, file names, an account name and chat IDs) and the descriptions of how the group works that our notes and hunt guide rely on | https://www.cisa.gov/news-events/cybersecurity-advisories/aa26-222a |
| **CISA's machine-readable copy of the same indicators** (STIX file, JSON) | Two extra hash values for one file (SHA-1 and MD5), which the page itself does not list | https://www.cisa.gov/sites/default/files/2026-08/AA26-222A-stix.json |

CISA states that content on its site is TLP:CLEAR unless labelled otherwise, which means it may be shared without
restriction, subject to copyright controls (https://www.cisa.gov/notification). We record the notice and our reading of
it in our own source notes.

## Further detail published by the same advisory (linked, not reproduced)

- The advisory's **MITRE ATT&CK mapping** of the group's methods, its **mitigations** and its **incident response**
  steps. The page also offers the indicators as downloadable files (STIX, in XML and JSON).
- **Four ransom-negotiation email addresses** (the advisory's Table 3). We did not copy them because our publishing
  checks do not allow email addresses in a public repository. Take them from the advisory.
- **A fourth chat ID.** The advisory lists four chat IDs. One of them contains a letter that cannot appear in such an ID,
  so it looks like a typing error in the advisory, and we did not copy it. Take it, corrected, from the advisory if it is
  updated.
- **Reporting.** The agencies ask victims to report to the FBI's Internet Crime Complaint Center, a local FBI or Secret
  Service field office, or CISA. The advisory lists the contact details.
