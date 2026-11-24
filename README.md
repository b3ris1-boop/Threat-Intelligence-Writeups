# Threat Intelligence Report: Multi-Stage Smishing & Telegram Scam Infrastructure ("Moxxie" Campaign)

## 📌 Executive Summary
On **November 24, 2026**, an unsolicited SMS phishing (Smishing) campaign originating from a Luxembourg VoIP number (`+352 33 66 45 10`) was observed targeting mobile users in Turkey. The campaign leveraged social engineering clickbait lures to drive victims through a multi-stage Traffic Distribution System (TDS) employing anti-analysis cloaking, custom cookie tracking, and mobile deep-linking to force interaction with a network of fraudulent Telegram bots (`Moxsie AI` / `Moxxie Ai` / `BUNKER AI`).

Through passive OSINT, sandbox detonation, certificate transparency logs, and WHOIS pivoting, the underlying infrastructure—active since **August 2026**—was mapped, attributed to a consistent affiliate/operator tag (`moxxie` / `moxxxie`), and reported to relevant registrar, CDN, and platform abuse teams.

---

## 🔍 Attack Chain & Technical Breakdown

### 1. Initial Access (Smishing Lure)
* **Delivery Vector:** Unsolicited SMS/MMS from `+35233664510` (Luxembourg country code).
* **Social Engineering Lure:** Turkish-language clickbait message claiming to offer non-consensual synthetic imagery ("deepnude") via Telegram to provoke curiosity and urgency.
* **Initial Payload URL:** `hxxp://t2r[.]pw/11`

### 2. Anti-Analysis Cloaking & Evasion
During initial command-line reconnaissance using `curl` with standard Linux/Android `User-Agent` headers, the server returned an `HTTP 200 OK` response serving a generic *"Invalid Link"* page.
* **Evasion Mechanism:** The redirector checks for full browser execution environments (JavaScript support, session cookies, and residential/specific ASN IP ranges) to block automated security crawlers and CLI tools.
* **Sandbox Bypass:** Detonating the URL inside an isolated cloud browser sandbox (**URLScan.io**, Netherlands exit node) successfully bypassed the cloaking filter and revealed the full HTTP redirection chain.

### 3. Multi-Stage Redirect Chain & Deep-Linking
Once browser validation checks passed, the infrastructure executed sequential `HTTP 307` and `HTTP 302` redirects:

1. **Stage 1 (Shortened Lure):** `hxxp://t2r[.]pw/11` (`HTTP 307` Internal Redirect)
2. **Stage 2 (Intermediary TDS):** `hxxps://moxxie[.]vip/go/moxxxie` (`HTTP 302`)
   * **Custom Session Tracking:** Sets a persistent tracking cookie named `mx_gb` (e.g., `1795521302702.cmw79hpp90034fmwvwl8dvnsi.1`) using a CUID-like identifier to track victim clicks and conversions.
3. **Stage 3 (Final C2 / Scam Endpoint):** `hxxps://t[.]me/videoFlash129_bot?start=moxxxie`
   * **Mobile App Triggering:** Executes a `tg://resolve?domain=videoFlash129_bot&start=moxxxie` URI scheme request to automatically launch the native Telegram application on mobile devices.

---<img width="1000" height="656" alt="image" src="https://github.com/user-attachments/assets/528fa790-a981-457d-a9d6-e3585c60cd6f" />


## 🕸️ Infrastructure Mapping & Attribution

Pivoting across WHOIS records, Certificate Transparency logs (`crt.sh`), and Telegram endpoints revealed a persistent, multi-bot operation rather than a single disposable setup:

* **Operator / Affiliate Attribution:** Consistent use of the `moxxie` / `moxxxie` identifier across the core domain (`moxxie[.]vip`), URL path (`/go/moxxxie`), Telegram referral parameter (`?start=moxxxie`), and bot display names (*Moxsie AI*, *Moxxie Ai*).
* **Domain Lifecycle & Registration (WHOIS):**
  * **Domain:** `moxxie[.]vip`
  * **Creation Date:** `2026-08-12T00:39:50Z` (Active for >3 months prior to this campaign wave)
  * **Registrar:** Dynadot Inc (IANA ID: 472)
  * **Name Servers:** `://cloudflare.com`, `://cloudflare.com`
* **SSL/TLS Certificate History (`crt.sh`):** Wildcard certificates (`*.moxxie[.]vip`) issued via Sectigo Limited (`2026-08-12`) and Google Trust Services (`2026-10-10`), confirming continuous infrastructure maintenance.
* **Network & Hosting Infrastructure:**
  * **CDN/Proxy Layer:** Cloudflare (`188.114.96.3`, `104.21.11.59` - AS13335)
  * **Associated Routing IP:** `45.9.148.208` — Managed by **Nice IT Customers Network (`NiceIT-NL` / AS49447)**, Netherlands.
* **Associated Telegram Bot Cluster:** Investigation of the final payload uncovered three interconnected scam/deepfake bots operating under the same network:
  1. `@videoFlash129_bot` (*Moxsie AI*) — Primary campaign target
  2. `@moxxieVideo988_bot` (*Moxxie Ai*) — Secondary/backup bot
  3. `@X_Bunkerbot` (*BUNKER AI*) — Affiliated ecosystem bot

---<img width="1000" height="176" alt="image" src="https://github.com/user-attachments/assets/24faf414-c78d-4118-b20d-949d19592801" />
<img width="634" height="275" alt="image" src="https://github.com/user-attachments/assets/741c99e2-9d8b-4710-9c93-6a445df4be58" />

<img width="619" height="98" alt="image" src="https://github.com/user-attachments/assets/2252d897-8ae4-4af5-9dac-f3bf5169fc0e" />

## 🛡️ Indicators of Compromise (IoCs)

*(Defanged to prevent accidental execution)*

| Indicator Type | Value | Role / Context |
| :--- | :--- | :--- |
| **Phone Number** | `+35233664510` | Smishing sender (Luxembourg VoIP) |
| **URL** | `hxxp://t2r[.]pw/11` | Stage 1 shortened SMS lure |
| **Domain** | `moxxie[.]vip` | Core Traffic Distribution System (TDS) |
| **URL** | `hxxps://moxxie[.]vip/go/moxxxie` | Stage 2 cloaked redirector path |
| **IP Address** | `45.9.148.208` | Associated infrastructure (`NiceIT-NL` / AS49447) |
| **Cookie** | `mx_gb` | Custom victim tracking cookie on `moxxie[.]vip` |
| **Telegram Bot** | `@videoFlash129_bot` | Primary scam endpoint (*Moxsie AI*) |
| **Telegram Bot** | `@moxxieVideo988_bot` | Affiliated bot (*Moxxie Ai*) |
| **Telegram Bot** | `@X_Bunkerbot` | Affiliated bot (*BUNKER AI*) |

---

## 🚀 Mitigation & Takedown Actions
Full technical evidence, sandbox redirect traces, and IoC lists were compiled and submitted to the relevant abuse teams:
* **Cloudflare Abuse:** Reported `moxxie[.]vip` for phishing/scam redirection behind CDN proxy.
* **Dynadot Abuse (`abuse@dynadot.com`):** Submitted registrar takedown request for `moxxie[.]vip` with sandbox verification logs.
* **Telegram Abuse (`abuse@telegram.org`):** Reported `@videoFlash129_bot` for facilitating non-consensual synthetic media and financial/account scams.
* **Hosting Provider (`abuse@as49447.net`):** Notified NiceIT-NL regarding malicious traffic routing on `45.9.148.208`.

---

## 🔗 References & Sandbox Artifacts
* **URLScan.io Public Detonation:** `https://urlscan.io/result/01a20d68-c293-7227-829d-81ac3a17a0ed/#summary`


---

## ⚠️ Disclaimer
*This investigation was conducted strictly using passive OSINT methodologies, public certificate/WHOIS registries, and isolated cloud sandbox environments. No active scanning, exploitation, or unauthorized interaction was performed against any target infrastructure.*
