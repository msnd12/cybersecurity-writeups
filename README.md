# Cybersecurity Portfolio: SOC and Phishing Analysis

Hands-on investigations from my cybersecurity studies at Nasser Centre for Science and Technology. Every write-up follows the same structure so it reads like an analyst's report:

**Objective, Tools used, Methodology (with screenshots), Commands and explanations, Findings, Mitigations, Lessons learned.**

## Skills demonstrated

- Phishing email triage: header analysis, SPF, DKIM and DMARC, content decoding and IOC extraction
- Malware analysis: static analysis of Office and PDF documents, sandboxing, hash and reputation lookups
- Network analysis: packet capture and PCAP investigation with tcpdump
- Tooling: CyberChef, VirusTotal, URLScan.io, Hybrid Analysis, ANY.RUN, MXToolbox, oledump, pdfid, pdf-parser, tcpdump, cut/sort/uniq, WHOIS
- Reporting: MITRE ATT&CK mapping, defanged indicators, evidence-based verdicts and recommended actions

## Write-ups

### Email analysis

| Write-up | Format | Focus |
| --- | --- | --- |
| [Email Header Analysis, Part 1: Lab Setup and Raw Headers](email-analysis/email-header-analysis-part-1-lab-setup-and-raw-headers.md) | Series, part 1 of 2 | Environment setup, raw-message inspection, sender IP and WHOIS lookup |
| [Email Header Analysis, Part 2: SPF, DKIM and DMARC Results (sample3.eml)](email-analysis/email-header-analysis-part-2-spf-dkim-dmarc.md) | Series, part 2 of 2 | Reading authentication results and DKIM alignment in a real message header |
| [Decoding Email Content, Part 1: Quoted-Printable (sample5.eml)](email-analysis/email-content-decoding-part-1-quoted-printable.md) | Series, part 1 of 2 | Locate and decode an embedded URL in a sample message |
| [Decoding Email Content, Part 2: MIME and Base64 (sample1.eml)](email-analysis/email-content-decoding-part-2-mime-and-base64.md) | Series, part 2 of 2 | Understanding MIME parts and decoding Base64 content to recover a URL |
| [Phishing Link Analysis, Part 1: Extracting URLs and IOCs](email-analysis/phishing-link-analysis-part-1-url-and-ioc-extraction.md) | Series, part 1 of 2 | Raw message source, CyberChef and the Email IOC Extractor |
| [Phishing Link Analysis, Part 2: Link Investigation Workflow](email-analysis/phishing-link-analysis-part-2-link-investigation-workflow.md) | Series, part 2 of 2 | CyberChef, URLScan.io, VirusTotal, URLVoid, PhishTank and Google Safe Browsing |

### Malware analysis

| Write-up | Format | Focus |
| --- | --- | --- |
| [Static Analysis of a Macro-Enabled Excel File and Two Suspicious PDFs](malware-analysis/maldoc-static-analysis-excel-and-pdf.md) | Standalone | Static analysis only: nothing was opened in Office or a PDF reader |
| [Malicious Word Document: Hybrid Analysis and ANY.RUN](malware-analysis/malicious-word-document-hybrid-analysis-and-anyrun.md) | Standalone | Static and dynamic analysis of BankPaymAdviceVend.Report.docx |
| [Email Attachment Analysis: ISO File, VirusTotal and Cisco Talos](malware-analysis/email-attachment-analysis-iso-virustotal-talos.md) | Standalone | Extracting, validating and corroborating indicators from a suspicious attachment |

### Network analysis

| Write-up | Format | Focus |
| --- | --- | --- |
| [tcpdump, Part 1: Capturing and Filtering Live Traffic](network-analysis/tcpdump-part-1-capturing-and-filtering.md) | Series, part 1 of 3 | Interfaces, netcat, a local web server, host and source filters |
| [tcpdump, Part 2: The TCP Handshake and a PCAP Investigation](network-analysis/tcpdump-part-2-tcp-handshake-and-pcap-investigation.md) | Series, part 2 of 3 | Capturing a netcat session, reading a saved capture and tracing a suspicious download |
| [tcpdump, Part 3: Triaging a PCAP and Finding SSLoad Traffic](network-analysis/tcpdump-part-3-pcap-triage-sslload.md) | Series, part 3 of 3 | Ranking top talkers with cut, sort and uniq, then tracing SSLoad C2, t.me lookups and a DLL download |

**Series** are written as parts that continue from one another. **Standalone** write-ups can be read on their own.

## Repository layout

- `email-analysis/`: headers, content decoding and phishing link analysis
- `malware-analysis/`: static and dynamic analysis of malicious documents and attachments
- `network-analysis/`: packet capture and traffic analysis
- `assets/`: screenshots used by the write-ups

*All work was done in isolated lab environments with sample data. Indicators are defanged.*
