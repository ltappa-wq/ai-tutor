# Data Model Spec: LMS Credentials & Connections

This specification defines the schema additions to support external LMS portal connections (Phase 4: Schoology crawl) using envelope encryption and session caching.

---

## 1. Schema Extensions (`lms_connections`)

The `lms_connections` table stores encrypted credentials and connection metadata. Each record belongs to exactly one `household`.

```sql
CREATE TYPE lms_provider_enum AS ENUM ('schoology', 'canvas', 'google_classroom');
CREATE TYPE lms_status_enum AS ENUM ('active', 'pending', 'invalid_credentials', 'reconnect_required', 'error', 'paused');

CREATE TABLE lms_connections (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    household_id UUID NOT NULL REFERENCES households(id) ON DELETE CASCADE,
    provider lms_provider_enum NOT NULL DEFAULT 'schoology',
    
    -- Connection Configuration
    portal_domain VARCHAR(255) NOT NULL,            -- e.g., "app.schoology.com" or "district.schoology.com"
    school_nid VARCHAR(64) NULL,                    -- School / building identifier if required by district SSO
    account_identifier_masked VARCHAR(255) NOT NULL, -- e.g., "j***n@gmail.com" for UI display
    
    -- Envelope Encryption Artifacts (AES-256-GCM)
    encrypted_dek BYTEA NOT NULL,                   -- KMS-encrypted Data Encryption Key
    encrypted_payload BYTEA NOT NULL,               -- AES-256-GCM ciphertext (contains username, password, metadata)
    iv BYTEA NOT NULL,                              -- 12-byte initialization vector (nonce)
    auth_tag BYTEA NOT NULL,                        -- 16-byte GCM authentication tag
    kms_key_id VARCHAR(255) NOT NULL,               -- ARN or Key ID of the KMS Master Key (KEK)
    aad_version INT NOT NULL DEFAULT 1,             -- Schema version for Additional Authenticated Data
    
    -- Execution State & Diagnostics
    status lms_status_enum NOT NULL DEFAULT 'pending',
    last_crawled_at TIMESTAMPTZ NULL,
    last_success_at TIMESTAMPTZ NULL,
    next_scheduled_crawl_at TIMESTAMPTZ NULL,
    consecutive_failures INT NOT NULL DEFAULT 0,
    last_error_code VARCHAR(64) NULL,               -- e.g., "AUTH_FAILED", "CAPTCHA_TRIGGERED", "DOM_TIMEOUT"
    last_error_message TEXT NULL,                   -- Sanitized, high-level error message safe for display
    
    -- Timestamps
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    CONSTRAINT uq_household_provider_domain UNIQUE (household_id, provider, portal_domain)
);

CREATE INDEX idx_lms_connections_crawl_schedule 
    ON lms_connections (status, next_scheduled_crawl_at) 
    WHERE status = 'active';
```

---

## 2. Session Cache Schema (`lms_sessions`)

To avoid decrypting master credentials for every single crawl, active session cookies are stored encrypted with a short expiration.

```sql
CREATE TABLE lms_sessions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    connection_id UUID NOT NULL REFERENCES lms_connections(id) ON DELETE CASCADE,
    
    -- Encrypted Session Cookies / State
    encrypted_session_data BYTEA NOT NULL,          -- AES-256-GCM encrypted cookie jar (JSON string)
    iv BYTEA NOT NULL,                              -- 12-byte IV
    auth_tag BYTEA NOT NULL,                        -- 16-byte auth tag
    
    -- Lifetime
    expires_at TIMESTAMPTZ NOT NULL,                -- Estimated session cookie expiration
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    
    CONSTRAINT uq_connection_session UNIQUE (connection_id)
);
```

---

## 3. Crawl Audit & Sync History (`lms_crawl_logs`)

Maintains an immutable record of sync operations for parent transparency, rate-limit tracking, and debugging.

```sql
CREATE TABLE lms_crawl_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    connection_id UUID NOT NULL REFERENCES lms_connections(id) ON DELETE CASCADE,
    household_id UUID NOT NULL REFERENCES households(id) ON DELETE CASCADE,
    
    started_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    completed_at TIMESTAMPTZ NULL,
    
    courses_detected INT NOT NULL DEFAULT 0,
    assignments_extracted INT NOT NULL DEFAULT 0,
    materials_ingested INT NOT NULL DEFAULT 0,
    
    status VARCHAR(32) NOT NULL,                    -- 'SUCCESS', 'PARTIAL', 'FAILED'
    error_summary TEXT NULL,
    crawl_duration_ms INT NULL
);

CREATE INDEX idx_lms_crawl_logs_household 
    ON lms_crawl_logs (household_id, started_at DESC);
```

---

## 4. Encryption Field Mappings & Payload Specification

### Plaintext JSON Structure (Pre-Encryption)
```json
{
  "username": "parent_user@example.com",
  "password": "plaintextPasswordHere",
  "portal_domain": "district.schoology.com",
  "school_nid": "987654321",
  "custom_headers": {}
}
```

### Additional Authenticated Data (AAD) Construction
The AAD byte sequence passed to AES-256-GCM must match exactly during both encryption and decryption:

$$	ext{AAD} = 	ext{"household\_id="} \parallel 	ext{connection.household\_id} \parallel 	ext{"|conn\_id="} \parallel 	ext{connection.id} \parallel 	ext{"|v=1"}$$

If the row is duplicated or transferred to another `household_id` or `connection.id`, GCM tag validation fails and decryption raises `AuthenticationError`.
