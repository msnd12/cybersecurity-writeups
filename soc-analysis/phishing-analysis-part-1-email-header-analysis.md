# Phishing Analysis, Part 1 of 2: Email Header Analysis

| | |
| --- | --- |
| **Format** | Series, part 1 of 2. Continues in [Part 2: URL and IOC Extraction](phishing-analysis-part-2-url-and-ioc-extraction.md) |
| **Category** | SOC analysis, phishing |
| **Environment** | Ubuntu analysis VM from the SOC 101 course labs |
| **Sample** | `sample1.eml` |
| **Tools** | Thunderbird, Sublime Text, terminal (`cat`, `whois`), MxToolbox Email Header Analyzer |

## Objective

Work out where a suspicious email really came from, whether it passed authentication, and whether the sender can be trusted, using only the message headers.

## Setup

1. Installed Thunderbird with `sudo apt install thunderbird` (it installs as a snap).
2. In Thunderbird, Settings, Privacy, turned on **Allow remote content in messages** so the sample would render the way a user would see it. This was done on a throwaway lab VM only. On a real investigation, remote content should stay blocked because it can trigger tracking beacons.
3. Installed Sublime Text, then added Package Control from the command palette (`Ctrl+Shift+P`).
4. The course files are organised by topic. Header samples live in `SOC101/course_files/01_Phishing_Analysis/01_Header_Analysis` and contain `sample1.eml`, `sample2.eml` and `sample3.eml`. This write-up covers `sample1.eml`.

## Analysis

### 1. Read the raw message

Printed the file in the terminal with `cat sample1.eml` and opened the same file in Sublime Text (`subl sample1.eml`), where it is easier to search and highlight.

### 2. Follow the Received chain

The message passes through Microsoft's Outlook front-end servers and arrives from `mail-40140.protonmail.ch` at IP **185.70.40.140**. Each hop is protected with TLS (TLS 1.2 and TLS 1.3 appear in the chain), and the timestamps are about 1 May 2024, 20:04 UTC.

### 3. Check the authentication results

| Check | Result | Detail |
| --- | --- | --- |
| SPF | pass | sender IP 185.70.40.140, `smtp.mailfrom=protonmail.com` |
| DKIM | timeout | key query timeout, `header.d=protonmail.com` |
| DMARC | pass | `action=none`, `header.from=protonmail.com` |
| Composite auth | pass | `compauth=pass reason=100` |

The `Received-SPF` header agrees: the domain protonmail.com designates 185.70.40.140 as a permitted sender.

### 4. Identify the sender IP

Searched for the word *sender* in Sublime to highlight the SPF line and confirm the sending IP: **185.70.40.140**.

### 5. WHOIS lookup

```
whois 185.70.40.140
```

The RIPE record shows the range `185.70.40.0 - 185.70.40.255`, netname `protonmail`, organisation **Proton Technologies AG**, country **CH**.

### 6. Cross-check with a header analyzer

Pasted the headers into the MxToolbox Email Header Analyzer. It summarised the subject (*Your Bank Account has been blocked due to unusual activities*), the SPF and DKIM alignment checks, and a relay table listing every hop with its delay, roughly 5 seconds from start to finish.

## Findings

- The mail was sent through Proton's real mail infrastructure. The infrastructure is legitimate, the message is not.
- SPF and DMARC pass for **protonmail.com**, the domain the attacker sends from. A pass tells you the sender is allowed to use that domain, not that the message comes from the bank it pretends to be.
- DKIM could not be verified because the key lookup timed out.
- The subject line is a classic urgency lure that impersonates a bank. Part 2 shows the displayed sender is a bank address while the reply path points to a free webmail account, which does not line up with the authenticated domain.

**Verdict:** phishing (brand impersonation).

## What I would do next in a SOC

These are my recommendations, not steps taken in the lab.

1. Quarantine the message and search for the same sender and subject across other mailboxes.
2. Block the sender address and the reply-to address at the mail gateway. Do not block the whole Proton IP range.
3. Extract and check the links and attachments, which is covered in Part 2.
4. Report the abuse to the provider using the contact in the WHOIS record.

## Lessons learned

- Read the Received chain from the bottom up. The lowest hop is closest to the real origin.
- A passing SPF or DMARC result only proves who sent the mail, not who it claims to be.
- Always compare the displayed From address with the Return-Path and Reply-To.
