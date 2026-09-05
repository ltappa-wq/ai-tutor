# Authentication & Credential Security Spec

This document specifies authentication, authorization, and credential vaulting for the AI Tutor platform across three trust tiers:
1. **Guardians (Parents):** Google SSO, household administration, LMS credential management.
2. **Students (Kids):** Parent-assigned username/passcode without email requirements.
3. **External LMS Portals (Schoology):** Zero-knowledge envelope encryption for headless crawling.

---

## 1. Guardian Authentication (Google SSO)

* **Identity Provider:** Google OAuth 2.0 / OpenID Connect (OIDC).
* **Scopes Requested:** `openid`, `email`, `profile`.
* **Household Association:**
  * The first guardian to register instantiates the `household` record and becomes the primary guardian.
  * Additional guardians join via cryptographically secure invitation tokens (`guardian_invites`), granting equal administrative rights.
* **Session Management:** Secure, HTTP-only, `SameSite=Lax`, partitioned session cookies with rolling 30-day expiration.

---

## 2. Student Authentication (Username & Passcode)

* **Design Rationale:** K–12 students often lack personal email addresses, or schools restrict external email delivery.
* **Credentials:**
  * `username`: Household-scoped or globally unique lowercase alphanumeric handle (assigned by guardian).
  * `passcode`: 4-to-6 digit numeric PIN or simple text passphrase (hashed with Argon2id or bcrypt, cost factor 12).
* **Session Isolation:**
  * Student sessions are strictly scoped to learning, chat, and materials review.
  * Students cannot view billing, guardian digest settings, or LMS connection secrets.
* **Account Recovery:**
  * Guardians can reset student passcodes instantly from the Parent Dashboard.

---

## 3. Schoology & External LMS Credential Vaulting

### A. Threat Model & Security Goals
Automating the 14:00 household-tz Schoology crawl requires storing parent portal credentials. If the application database is compromised, plaintext or reversibly encrypted passwords must **not** be extractable.

* **Goal 1: Encryption in Transit & Rest:** Credential payloads are never stored in plaintext and never logged in traces, APMs, or error messages.
* **Goal 2: Asymmetric Privilege Separation:** The public-facing Web API can **only encrypt** credentials; it has **no permission to decrypt** them.
* **Goal 3: Isolated Decryption Boundary:** Only the background crawler worker running in an isolated execution environment possesses KMS decryption privileges.
* **Goal 4: Cryptographic Tenant Isolation:** Ciphertext is cryptographically bound to the specific `household_id` using Authenticated Encryption with Associated Data (AEAD) to prevent tenant-swapping attacks.

---

### B. Cryptographic Architecture: Envelope Encryption

```
                    ┌─────────────────────────┐
                    │ Cloud KMS (AWS/GCP/Vault)│
                    │  Master Key (KEK) (HSM) │
                    └────────────┬────────────┘
                                 │
           1. GenerateDataKey()  │  4. Decrypt(Encrypted DEK)
                                 │
    ┌────────────────────────────┴────────────────────────────┐
    ▼                                                         ▼
┌───────────────────────────────┐         ┌───────────────────────────────┐
│        Web / API Server       │         │     Crawler Worker (14:00)    │
│  (Role: kms:GenerateDataKey)  │         │      (Role: kms:Decrypt)      │
│                               │         │                               │
│  - Receives password via POST │         │  - Fetches ciphertext & DEK   │
│  - Calls KMS GenerateDataKey  │         │  - Decrypts DEK via KMS       │
│  - Encrypts via AES-256-GCM   │         │  - Decrypts payload in-memory │
│  - Zeroes Plaintext Key & Pass│         │  - Executes Schoology session │
│  - Saves Ciphertext to DB     │         │  - Zeroes credentials/DEK     │
└───────────────┬───────────────┘         └───────────────▲───────────────┘
                │                                         │
                │ 2. Insert record                        │ 3. Fetch record
                ▼                                         │
       ┌──────────────────────────────────────────────────┴┐
       │             PostgreSQL Database Table             │
       │               `lms_connections`                   │
       └───────────────────────────────────────────────────┘
```

#### 1. Key Hierarchy
* **Key Encryption Key (KEK):** Managed inside Cloud Key Management Service (AWS KMS, GCP Cloud KMS, or HashiCorp Vault). Configured with automatic annual rotation. Never leaves the hardware security module (HSM).
* **Data Encryption Key (DEK):** A single-use 256-bit symmetric key generated dynamically per credential record via KMS `GenerateDataKey(KeyId=KEK, KeySpec=AES_256)`.

#### 2. Cipher & Authenticated Data (AEAD)
* **Cipher Suite:** `AES-256-GCM` (Galois/Counter Mode).
* **Initialization Vector (IV / Nonce):** 96-bit (12-byte) cryptographically secure random bytes generated uniquely per encryption. **Never reused.**
* **Authentication Tag:** 128-bit (16-byte) authentication tag enforcing data integrity and authenticity.
* **Additional Authenticated Data (AAD):**
  ```text
  AAD = "household_id=" + household_id + "|service=schoology|v=1"
  ```
  *Crucial:* AES-GCM validates that the ciphertext belongs exclusively to that specific household. If a malicious actor copies encrypted blobs between database rows, decryption will fail with an authentication error.

#### 3. Payload Structure (Pre-Encryption)
Before encryption, the payload is serialized as strict JSON:
```json
{
  "auth_type": "credentials",
  "domain": "district.schoology.com",
  "school_nid": "12345678",
  "username": "guardian@example.com",
  "password": "CorrectHorseBatteryStaple123!",
  "created_at": "2026-09-04T05:00:00Z"
}
```

---

### C. In-Memory Lifetime & Hygiene

1. **Immediate Zeroization:**
   * After the Web API encrypts the payload, buffers containing the plaintext password and plaintext DEK are explicitly overwritten in memory (e.g., `buffer.fill(0)` or platform-specific secure wipe) before garbage collection.
   * The crawler worker decrypts credentials strictly into memory for the duration of the HTTP/browser authentication handshake, then scrubs the buffer immediately once session cookies are obtained.
2. **Session Token Caching (Minimizing Decryption Frequency):**
   * Upon successful login, the crawler stores encrypted Schoology session cookies (`ASP.NET_SessionId`, `SessID`, etc.) with a short Time-To-Live (e.g., 7 days).
   * Subsequent daily crawls use the cached session cookies first. The master credentials are only re-decrypted if the session returns a `401 Unauthorized` or redirect to `/login`.
3. **Log Sanitization:**
   * Global interceptor blocks keys matching `password`, `passcode`, `token`, `secret`, `encrypted_payload`, or `encrypted_dek` from application logs, APM traces, and Sentry error captures.

---

### D. IAM Role & Access Separation

| Service Component | Allowed KMS Actions | Database Permissions |
| :--- | :--- | :--- |
| **Web / Backend API** | `kms:GenerateDataKey`, `kms:Encrypt` | `INSERT`, `UPDATE`, `SELECT` on `lms_connections` |
| **Crawler Background Worker** | `kms:Decrypt` | `SELECT`, `UPDATE (status, last_crawled_at)` on `lms_connections` |
| **Database Administrator** | *None* | Table maintenance only (cannot decrypt without KMS role) |

*The Web API cannot decrypt stored credentials even if compromised via remote code execution.*

---

### E. Error Handling & Re-Authentication Flows

* **Invalid Credentials:** If Schoology rejects credentials (e.g., password changed by parent), set status to `invalid_credentials` and notify the household via the Parent Dashboard and nightly digest.
* **CAPTCHA / 2FA Challenge:** If Schoology triggers an anti-bot challenge:
  1. Terminate the headless runner cleanly without retrying in a tight loop (to avoid account lockouts).
  2. Mark status as `reconnect_required`.
  3. Surface an alert on the parent home screen: *"Schoology connection paused. Tap to reconnect or upload course materials manually."*
  4. Preserve upload-first ingestion as an uninterrupted fallback.
