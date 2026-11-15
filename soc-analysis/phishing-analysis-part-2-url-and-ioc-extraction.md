# Phishing Analysis, Part 2 of 2: URL and IOC Extraction

| | |
| --- | --- |
| **Format** | Series, part 2 of 2. Continues from [Part 1: Email Header Analysis](phishing-analysis-part-1-email-header-analysis.md) |
| **Category** | SOC analysis, phishing |
| **Environment** | Ubuntu analysis VM from the SOC 101 course labs |
| **Sample** | `sample1.eml`, from the `03_URL_Analysis` folder |
| **Tools** | Thunderbird, CyberChef, Email-IOC-Extractor (`eioc.py`), `wget` |

## Objective

Pull every URL out of the phishing email from Part 1, make the links safe to share, and collect the indicators of compromise (IOCs) in one place.

## Analysis

### 1. Look at the email as the user would

Opened `sample1.eml` in Thunderbird. The message is styled as a bank notice with the subject *Your Bank Account has been blocked due to unusual activities*. It tells the reader their account is suspended and shows a **Reactivate Your Account** button.

### 2. Read the message source

In Thunderbird, **View, Message Source** (`Ctrl+U`) shows the raw HTML. I searched the source for the strings below.

| Search term | What it found |
| --- | --- |
| `http` | An image `src` pointing at `raw.githubusercontent.com`, used for the page graphics |
| `<a class` | The link behind the **Reactivate Your Account** button, which points at a link-shortening domain (`dsgo.to`) and not at the bank |
| `quoted` | The header `Content-Transfer-Encoding: quoted-printable` |

The last result explains why the URLs look broken in the raw source. In quoted-printable text, `=3D` stands for `=` and lines end with a soft-break `=`, so long URLs are chopped into pieces.

### 3. Decode and extract in CyberChef

Copied the whole message source into CyberChef and built a three-step recipe:

1. **From Quoted Printable** to undo the encoding
2. **Extract URLs**
3. **Defang URL** with *Escape dots*, *Escape http* and *Escape ://* ticked

The output contained two URLs, both safe to paste into a ticket because they cannot be clicked by accident:

```
hxxps[://]raw[.]githubusercontent[.]com/MalwareCube/SOC101/main/assets/01_Phishing_Analysis/...jpg
hxxps[://]dsgo[.]to/<long tracking token>
```

### 4. Automate it with Email-IOC-Extractor

The same job can be done in one command with the open source Email-IOC-Extractor script from MalwareCube.

```
wget https://github.com/MalwareCube/Email-IOC-Extractor/blob/main/eioc.py
```

This first attempt failed with `Cannot write to 'eioc.py' (Permission denied)` because I was in a protected directory. Running it again with `sudo` saved the file, and I then ran the script against `sample1.eml`.

### 5. Results

The script prints defanged IPs and URLs and pulls out the most useful headers.

| Indicator | Value |
| --- | --- |
| Sender IP | `185[.]70[.]40[.]140` (Plan-les-Ouates, Geneva, CH, AS62371 Proton AG) |
| Internal relay IP | a private `10.x` address from the Microsoft relay hop |
| URL 1 | the `raw.githubusercontent.com` image link (defanged) |
| URL 2 | the `dsgo.to` redirect link (defanged) |
| From (displayed) | a bank address, `alerts@chase.com` |
| Reply-To and Return-Path | a free ProtonMail address |
| X-Sender-IP | `185.70.40.140` |
| Date | Wed, 01 May 2024 20:04:05 +0000 |
| Attachments | none reported |

## Findings

- The only link a user would click goes through a **URL shortener**, so the real destination is hidden from the reader.
- The displayed sender is a bank, but replies and bounces go to a free webmail account. This confirms the impersonation noted in Part 1.
- The email contains **no attachments**, so the attack depends entirely on the link.

**Verdict:** credential-phishing email. The link should be blocked and the message removed from all mailboxes.

## What I would do next in a SOC

These are recommendations, not steps taken in the lab.

1. Submit the defanged `dsgo.to` link to a sandbox or URL scanner such as urlscan.io or VirusTotal to see where it redirects. Never open it directly.
2. Add the redirect domain and the reply-to address to the block list.
3. Search email logs for other messages containing the same link.

## Lessons learned

- Always decode the message first. Quoted-printable and base64 bodies hide and break URLs.
- Defang every IOC before it goes into a ticket or chat (`hxxps`, `[.]`).
- A permission error when saving a file is a hint that you are in a protected folder, not that the download failed.
- Scripts like the IOC extractor give the same answer as a manual CyberChef recipe in seconds, so they are worth keeping in the toolbox.
