# 🏥 Mediroza General Hospital — Black-Box Penetration Test

**Week 4 of the Network Walks Cybersecurity Internship — full web application penetration test, from unauthenticated attacker to patient records, payroll and shareholder data**

![Skill-Cybersecurity](https://img.shields.io/badge/Skill-Cybersecurity-404040?style=flat-square&labelColor=C00000)
![Type-BlackBox](https://img.shields.io/badge/Test-Black--Box-6A1B9A?style=flat-square&labelColor=C00000)
![Target](https://img.shields.io/badge/Target-medirozahospital.com-1565C0?style=flat-square&labelColor=C00000)
![Findings](https://img.shields.io/badge/Findings-18-C62828?style=flat-square&labelColor=C00000)
![Critical](https://img.shields.io/badge/Critical-1-8B0000?style=flat-square&labelColor=C00000)
![Report](https://img.shields.io/badge/Report-28%20pages-2E7D32?style=flat-square&labelColor=C00000)
![NetworkWalks](https://img.shields.io/badge/NetworkWalks-404040?style=flat-square&labelColor=C00000)

---

## 📌 Project Overview

This repository contains my **Week 4** submission for the Network Walks Cybersecurity Internship: a
black-box penetration test of **Mediroza General Hospital** (`https://medirozahospital.com`).

The engagement followed the four project milestones — initial access, data extraction, critical data
exposure and reporting — and ended with a complete, professional penetration testing report.

| Milestone | Objective | Outcome |
| --- | --- | --- |
| **M1** | Attack the website and retrieve 3 confidential patient PDF lab reports | ✅ SQL injection → authentication bypass → 3 reports downloaded |
| **M2** | Crack the encryption on all 3 retrieved files | ✅ All 3 passwords recovered with John the Ripper + rockyou |
| **M3** | Find staff salaries and shareholder details | ✅ PDF metadata → exposed DB backup → full payroll + shareholder register |
| **M4** | Write a professional penetration testing report | ✅ 28-page report (docx + pdf) |

---

## 🎯 Scope & Authorization

| Item | Detail |
| --- | --- |
| Target | `https://medirozahospital.com` (199.188.201.16) |
| Type | Black-box web application penetration test |
| Authorization | **Written permission granted** by Network Walks for the internship project |
| Rules | Target domain only · no social engineering · no denial of service · no data modification |
| Window | 22 November 2026 |

> This project was conducted in a controlled environment for educational purposes only. These
> techniques must never be applied to any system without explicit written permission from the owner.

---

## 🔗 Attack Chain

```
robots.txt  ──►  /old/  /patient/  /staff/
                     │
                     ▼
        /patient/ directory listing
                     │
                     ▼
   SQL injection in login.php  ──►  authentication bypass
                     │
                     ▼
        patient portal  ──►  3 encrypted PDF reports
                     │
                     ▼
   $pdf$ hashes  ──►  John + rockyou  ──►  3 passwords
                     │
                     ▼
   PDF metadata: "DB backup moved to /old"
                     │
                     ▼
   /old/mediroza_db_backup_2019.sql
                     │
                     ▼
   staff salaries + national IDs + shareholder register
```

---

## 🔍 Findings Summary

| ID | Finding | Severity | CVSS |
| --- | --- | --- | --- |
| F-01 | SQL injection in patient login → authentication bypass | **Critical** | 9.1 |
| F-02 | Publicly exposed database backup (PII, salaries, shareholders) | **High** | 7.5 |
| F-03 | Weak password-based encryption protecting patient PDF reports | Medium | 6.5 |
| F-04 | Directory listing enabled on `/patient/`, `/staff/`, `/old/` | Medium | 5.3 |
| F-05 | Sensitive internal note stored in PDF metadata | Medium | 4.3 |
| F-06 | Application error log exposed in the web root | Medium | 4.3 |
| F-07 | Missing HTTP security headers | Low | 3.7 |
| F-08 | Technology and version disclosure | Low | 3.7 |
| F-09 | Session cookie without HttpOnly / SameSite | Low | 3.1 |
| F-10 | Username enumeration through distinct login errors | Medium | 5.3 |
| F-11 | Session fixation enabling account takeover | **High** | 8.1 |
| F-12 | Verbose SQL errors echoed to the browser | Medium | 5.3 |
| F-13 | E-mail domain spoofable (SPF softfail, DMARC `p=none`) | Medium | 5.3 |
| F-14 | Undocumented look-alike login page on a subdomain | Medium | 6.1 |
| F-15 | No CAA record and no DNSSEC | Low | 3.7 |
| F-16 | Mail server build covered by a current Exim advisory | **High** | 8.1 |
| F-17 | Certificate name mismatch on mail and control-panel hosts | Medium | 4.2 |
| F-18 | Outdated nginx base in the edge/WAF tier | Medium | 5.9 |

**Overall risk: CRITICAL.** An unauthenticated attacker can reach patient records, the full staff
payroll and the shareholder register in a handful of requests.

**Severity mix:** 1 Critical · 3 High · 10 Medium · 4 Low (18 findings).

---

## 🔑 Recovered Passwords

| Report | Patient | Password | Method |
| --- | --- | --- | --- |
| `patient_report_1.pdf` | Sipho Dlamini | `123456` | rockyou + six-digit mask |
| `patient_report_2.pdf` | Priya Reddy | `password` | rockyou |
| `patient_report_3.pdf` | Emily Thompson | `!@#$%^&` | rockyou |

All three files used the standard PDF security handler (V2 / R3, 128-bit RC4). Hashes were rebuilt
from the encryption dictionaries in the format:

```
$pdf$<V>*<R>*<keylen>*<P>*<EncryptMetadata>*<id_len>*<id>*<u_len>*<u>*<o_len>*<o>
```

and cracked with John the Ripper 1.9.0-jumbo against `rockyou.txt` in about two seconds.

---

## 🧪 Extra Testing (beyond the milestones)

- **Content discovery** — 37 sensitive paths (backups, `.git`, `.env`, config files, admin panels,
  upload directories, `server-status`); no further exposure found.
- **HTTP methods** — `TRACE`, `PUT`, `DELETE` all return 501; `OPTIONS` advertises only
  `OPTIONS, HEAD, GET, POST`.
- **Transport security** — TLS 1.3 / AES-256-GCM, valid certificate, HTTP → HTTPS 301.
- **Session handling** — logout invalidates the session; the session ID is **not** rotated on login
  (F-11), and an attacker-chosen identifier is accepted and adopted after login.
- **User enumeration** — "Incorrect password" vs "Username not found" reveals valid accounts.
- **Second cracking approach** — a pure six-digit mask attack recovered `123456` in 12 seconds.
- **Anti-automation** — an openresty JavaScript challenge engages on automated traffic, but only
  after the exploitation had already completed.
- **Positive controls** — staff login is not injectable, `download.php` resists path traversal,
  patient reports are served with `Cache-Control: no-store`.

### System-level research (the deeper pass)

- **DNS and e-mail authentication** — SPF ends in `~all` and DMARC is `p=none`, so the domain can be
  spoofed for patient-directed phishing; no CAA record and no DNSSEC (F-15, F-13).
- **Certificate Transparency** — the CT logs revealed an unlinked subdomain,
  `heartpatient.medirozahospital.com`, serving a look-alike patient login page that asks for
  "your portal username and password" and claims credentials travel in clear text (F-14).
- **Service inventory** — 46-name DNS dictionary, then banner grabs: Exim 4.100.1 on 587, Dovecot
  on 993/995, Pure-FTPd on 21, cPanel/WHM/webmail vhosts, plus the transport-level certificate
  matrix (apex and SMTP certs match; mail/cpanel/webmail/ftp serve the `*.web-hosting.com` wildcard).
- **CVE correlation** — every identified version matched against current advisories:
  - PHP 8.2.36 is the **patched** release from 19 Nov 2026 (fixes phpAfterShell,
    CVE-2026-129289 and 15 others) → not vulnerable.
  - Exim 4.100.1 is one advisory behind: EXIM-Security-2026-11-19.1 (heap OOB write in SMTP
    reception, High, fixed in 4.100.2) was published on 25 Nov 2026 during the assessment and covers
    the installed build → **F-16**.
  - OpenResty 1.31.1.1 (edge/WAF) bundles nginx 1.31.1, which falls in the affected range of several
    2026 nginx CVEs fixed in 1.31.9 / 1.30.7; HTTP/2 is negotiated via ALPN, so this is tracked as
    **F-18** (no memory-corruption or DoS payload was sent).
  - LiteSpeed build number hidden — flagged for hosting-provider confirmation.
- **Protocol posture** — TLS 1.3 + TLS 1.2 with forward secrecy, ALPN `h2`, HTTP/3 not advertised,
  unknown `Host` headers fall through to the cPanel default vhost.
- **Injection depth** — the app echoes raw `mysqli_query()` errors including the failing SQL
  fragment, which is its own finding (F-12); `download.php` accepts only 1–3 and rejects every
  other value identically.

---

## 🛠️ Tools Used

| Tool | Purpose |
| --- | --- |
| curl / Python `requests` | HTTP interaction, header inspection, request replay |
| Chrome (browser) | Visual confirmation and screenshot evidence |
| John the Ripper 1.9.0-jumbo | Dictionary and mask attacks against PDF hashes |
| rockyou.txt | Password candidate source |
| pypdf / PyMuPDF | PDF decryption, metadata extraction, rendering |
| ExifTool | File and encryption metadata inspection |
| Python (custom) | `$pdf$` hash builder and evidence collection |

---

## 📁 Repository Structure

```
Week Four Submissions/
├── W4-PENTEST-REPORT - Mediroza General Hospital.pdf
├── README.md
└── Evidence/
    ├── 01-http-headers.txt
    ├── 02-login-sqli.txt
    ├── 03-portal-access.txt
    ├── 04-directory-listings.txt
    ├── 05-db-backup.txt
    ├── 06-pdf-metadata.txt
    ├── 07-session-and-enumeration.txt
    ├── 08-john-crack.txt
    ├── 09-tls.txt
    ├── 10-content-discovery.txt
    ├── 11-mask-attack.txt
    ├── 12-waf-challenge.html
    ├── 13-system-and-cve-research.txt | DNS, e-mail authentication, CT logs, subdomains, service banners and CVE
    ├── 14-app-layer-verification.txt | Security headers, clickjacking PoC, session-fixation proof, throttling
    ├── mediroza_db_backup_2019.sql
    ├── show_1.txt / show_2.txt / show_3.txt
    ├── PDFs/           (3 original encrypted + 3 decrypted reports)
    └── Screenshots/    (browser and document evidence, incl. clickjacking PoC)
```

---

## 🔁 Reproducing the Key Steps

```bash
# 1. Authentication bypass
curl -s -i -d "username=admin'-- -&password=x" \
     https://medirozahospital.com/patient/login.php

# 2. Report retrieval (authenticated session)
curl -s -b cookies.txt -o patient_report_1.pdf \
     "https://medirozahospital.com/patient/download.php?id=1"

# 3. Password recovery
john --format=pdf --wordlist=rockyou.txt hash_1.txt

# 4. Exposed backup
curl -s -O https://medirozahospital.com/old/mediroza_db_backup_2019.sql
```

---

## 📚 What I Learned

1. **Input handling is still the front door.** One unescaped quote turned into full portal access.
2. **Metadata is data.** A note in a PDF's `/Comments` field pointed straight at the worst exposure.
3. **Backups are the real crown jewels.** The database dump was worth more than the application.
4. **Encryption without a strong password is theatre.** Three reports fell in seconds.
5. **Small details matter.** Cookie flags, error messages, directory indexes and headers each widen
   the attack surface on their own.

---

## ⚠️ Disclaimer

All testing was performed against an authorised target as part of the Network Walks Cybersecurity
Internship. The techniques documented here are for educational purposes only and must never be used
against systems without explicit written permission from the owner.
