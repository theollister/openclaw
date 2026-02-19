# OpenClaw Security Architecture

## Security Layers

```mermaid
graph TB
    subgraph "Network Security"
        TLS[TLS/HTTPS]
        FIREWALL_SEC[Firewall]
        VPN[VPN/Tailscale]
        RATELIMIT[Rate Limiting]
    end
    
    subgraph "Authentication"
        OAUTH_SEC[OAuth 2.0]
        APIKEYS_SEC[API Key Management]
        TOKENS[JWT Tokens]
        REFRESH[Token Refresh]
    end
    
    subgraph "Authorization"
        ALLOWLIST[Allowlists]
        PAIRING_SEC[Device Pairing]
        PERMISSIONS[Permissions]
        RBAC[Role-Based Access]
    end
    
    subgraph "Data Security"
        ENCRYPTION[Encryption at Rest]
        KEYCHAIN_SEC[Keychain/KeyStore]
        SECRETS_SEC[Secret Management]
        REDACTION[PII Redaction]
    end
    
    subgraph "Application Security"
        INPUT[Input Validation]
        SANITIZE[Sanitization]
        XSRF[CSRF Protection]
        INJECTION[Injection Prevention]
    end
    
    TLS --> OAUTH_SEC
    FIREWALL_SEC --> RATELIMIT
    VPN --> ALLOWLIST
    
    OAUTH_SEC --> TOKENS
    TOKENS --> REFRESH
    APIKEYS_SEC --> SECRETS_SEC
    
    ALLOWLIST --> PERMISSIONS
    PAIRING_SEC --> RBAC
    
    ENCRYPTION --> KEYCHAIN_SEC
    KEYCHAIN_SEC --> SECRETS_SEC
    SECRETS_SEC --> REDACTION
    
    INPUT --> SANITIZE
    SANITIZE --> XSRF
    XSRF --> INJECTION
    
    style TLS fill:#51cf66
    style ENCRYPTION fill:#ff6b6b
    style ALLOWLIST fill:#4ecdc4
```

## Device Pairing Security

```mermaid
sequenceDiagram
    participant Primary as Primary Device
    participant Code as Pairing Code
    participant Secondary as Secondary Device
    participant Crypto
    
    Primary->>Primary: Generate pairing code
    Primary->>Crypto: Create key pair
    Crypto-->>Primary: Public + Private keys
    
    Primary->>Code: Display pairing code + QR
    Code->>Code: Time-limited (5 min)
    
    Secondary->>Code: Scan QR or enter code
    Secondary->>Primary: Request pairing
    
    Primary->>Primary: Verify code
    Primary->>Primary: User confirms on primary
    
    alt Confirmed
        Primary->>Crypto: Generate session key
        Crypto-->>Primary: Session key
        
        Primary->>Secondary: Send encrypted config
        Secondary->>Crypto: Decrypt with session key
        Crypto-->>Secondary: Config data
        
        Secondary->>Secondary: Store credentials
        Primary->>Primary: Mark device as paired
    else Denied
        Primary->>Secondary: Reject pairing
        Secondary->>Secondary: Show error
    end
```

## Credential Storage

```mermaid
graph TB
    subgraph "Credential Types"
        OAUTH_CREDS[OAuth Tokens]
        API_CREDS[API Keys]
        CHANNEL_CREDS[Channel Tokens]
        SIGNING[Signing Keys]
    end
    
    subgraph "Storage Mechanism"
        PLATFORM{Platform?}
    end
    
    subgraph "macOS"
        KEYCHAIN_MAC[macOS Keychain]
        ACL_MAC[Access Control List]
        SECURE_ENCLAVE[Secure Enclave]
    end
    
    subgraph "iOS"
        KEYCHAIN_IOS[iOS Keychain]
        BIOMETRIC[Biometric Auth]
        DATA_PROTECTION[Data Protection]
    end
    
    subgraph "Android"
        KEYSTORE[Android KeyStore]
        BIOMETRIC_ANDROID[Biometric Auth]
        ENCRYPTION_ANDROID[Hardware Encryption]
    end
    
    subgraph "Linux/Pi"
        FILE_ENCRYPTED[Encrypted Files]
        FILE_PERMS[File Permissions 0600]
        CRYPTO_LIB[libsodium/OpenSSL]
    end
    
    OAUTH_CREDS --> PLATFORM
    API_CREDS --> PLATFORM
    CHANNEL_CREDS --> PLATFORM
    SIGNING --> PLATFORM
    
    PLATFORM -->|macOS| KEYCHAIN_MAC
    PLATFORM -->|iOS| KEYCHAIN_IOS
    PLATFORM -->|Android| KEYSTORE
    PLATFORM -->|Linux/Pi| FILE_ENCRYPTED
    
    KEYCHAIN_MAC --> ACL_MAC
    KEYCHAIN_MAC --> SECURE_ENCLAVE
    
    KEYCHAIN_IOS --> BIOMETRIC
    KEYCHAIN_IOS --> DATA_PROTECTION
    
    KEYSTORE --> BIOMETRIC_ANDROID
    KEYSTORE --> ENCRYPTION_ANDROID
    
    FILE_ENCRYPTED --> FILE_PERMS
    FILE_ENCRYPTED --> CRYPTO_LIB
    
    style KEYCHAIN_MAC fill:#4ecdc4
    style KEYSTORE fill:#95e1d3
    style FILE_ENCRYPTED fill:#ffd93d
```

## Allowlist & Authorization

```mermaid
graph LR
    subgraph "Incoming Request"
        REQUEST[Request]
        SENDER_ID[Sender ID]
        CHANNEL_ID[Channel Type]
        COMMAND[Command/Action]
    end
    
    subgraph "Allowlist Checks"
        CHECK_SENDER[Check Sender Allowlist]
        CHECK_CHANNEL[Check Channel Allowlist]
        CHECK_COMMAND[Check Command Authorization]
    end
    
    subgraph "Decision"
        ALLOW_SEC[Allow]
        DENY_SEC[Deny]
        LOG_ATTEMPT[Log Attempt]
        NOTIFY_ADMIN[Notify Admin]
    end
    
    REQUEST --> CHECK_SENDER
    SENDER_ID --> CHECK_SENDER
    CHANNEL_ID --> CHECK_CHANNEL
    COMMAND --> CHECK_COMMAND
    
    CHECK_SENDER -->|Allowed| CHECK_CHANNEL
    CHECK_SENDER -->|Blocked| DENY_SEC
    
    CHECK_CHANNEL -->|Allowed| CHECK_COMMAND
    CHECK_CHANNEL -->|Blocked| DENY_SEC
    
    CHECK_COMMAND -->|Authorized| ALLOW_SEC
    CHECK_COMMAND -->|Unauthorized| DENY_SEC
    
    DENY_SEC --> LOG_ATTEMPT
    LOG_ATTEMPT --> NOTIFY_ADMIN
    
    style ALLOW_SEC fill:#51cf66
    style DENY_SEC fill:#ff6b6b
    style CHECK_SENDER fill:#4ecdc4
```

## Encryption Flow

```mermaid
graph TB
    subgraph "Data at Rest"
        CONFIG_FILE[Config Files]
        CREDS_FILE[Credential Files]
        SESSION_FILE[Session Files]
        LOGS_FILE[Log Files]
    end
    
    subgraph "Encryption Process"
        MASTER_KEY[Master Key]
        DERIVE[Key Derivation]
        ENCRYPT_DATA[Encrypt Data]
        STORE_ENCRYPTED[Store Encrypted]
    end
    
    subgraph "Decryption Process"
        RETRIEVE[Retrieve Encrypted]
        DECRYPT_DATA[Decrypt Data]
        VALIDATE[Validate Integrity]
        USE[Use in Memory]
    end
    
    subgraph "Key Management"
        GENERATE[Generate Key]
        ROTATE_KEY[Rotate Key]
        BACKUP[Secure Backup]
        DESTROY[Secure Deletion]
    end
    
    CONFIG_FILE --> ENCRYPT_DATA
    CREDS_FILE --> ENCRYPT_DATA
    SESSION_FILE --> ENCRYPT_DATA
    LOGS_FILE --> ENCRYPT_DATA
    
    MASTER_KEY --> DERIVE
    DERIVE --> ENCRYPT_DATA
    ENCRYPT_DATA --> STORE_ENCRYPTED
    
    STORE_ENCRYPTED --> RETRIEVE
    RETRIEVE --> DECRYPT_DATA
    DECRYPT_DATA --> VALIDATE
    VALIDATE --> USE
    
    MASTER_KEY --> GENERATE
    GENERATE --> ROTATE_KEY
    ROTATE_KEY --> BACKUP
    BACKUP --> DESTROY
    
    style MASTER_KEY fill:#ff6b6b
    style ENCRYPT_DATA fill:#4ecdc4
    style VALIDATE fill:#51cf66
```

## Secure Communication

```mermaid
sequenceDiagram
    participant Client
    participant Gateway
    participant Provider
    participant Channel
    
    Client->>Gateway: HTTPS Request (TLS 1.3)
    Gateway->>Gateway: Verify client certificate
    Gateway->>Gateway: Check allowlist
    
    alt Authorized
        Gateway->>Provider: API Request (TLS)
        Provider->>Provider: Validate request signature
        Provider-->>Gateway: Encrypted response
        
        Gateway->>Channel: Webhook (TLS)
        Channel->>Channel: Verify webhook signature
        Channel-->>Gateway: Acknowledge
        
        Gateway->>Gateway: Encrypt sensitive data
        Gateway-->>Client: Encrypted response (HTTPS)
        Client->>Client: Decrypt & validate
    else Unauthorized
        Gateway->>Gateway: Log unauthorized attempt
        Gateway-->>Client: 403 Forbidden
    end
```

## Vulnerability Protection

```mermaid
graph TB
    subgraph "Input Threats"
        INJECTION_THREAT[SQL/Command Injection]
        XSS[Cross-Site Scripting]
        CSRF[Cross-Site Request Forgery]
        PATH_TRAVERSAL[Path Traversal]
    end
    
    subgraph "Protection Mechanisms"
        VALIDATION[Input Validation]
        PARAMETERIZED[Parameterized Queries]
        ESCAPE[Output Escaping]
        TOKENS_CSRF[CSRF Tokens]
        SANDBOX_SEC[Sandboxing]
    end
    
    subgraph "Monitoring"
        IDS[Intrusion Detection]
        ANOMALY[Anomaly Detection]
        AUDIT_SEC[Security Audit Logs]
        ALERTS_SEC[Alert System]
    end
    
    subgraph "Response"
        BLOCK[Block Request]
        QUARANTINE[Quarantine User]
        INCIDENT[Incident Response]
        PATCH[Security Patching]
    end
    
    INJECTION_THREAT --> VALIDATION
    XSS --> ESCAPE
    CSRF --> TOKENS_CSRF
    PATH_TRAVERSAL --> SANDBOX_SEC
    
    VALIDATION --> PARAMETERIZED
    ESCAPE --> IDS
    TOKENS_CSRF --> IDS
    SANDBOX_SEC --> IDS
    
    IDS --> ANOMALY
    ANOMALY --> AUDIT_SEC
    AUDIT_SEC --> ALERTS_SEC
    
    ALERTS_SEC --> BLOCK
    ALERTS_SEC --> QUARANTINE
    ALERTS_SEC --> INCIDENT
    INCIDENT --> PATCH
    
    style VALIDATION fill:#4ecdc4
    style IDS fill:#ff6b6b
    style BLOCK fill:#ffd93d
```

## Privacy & Data Handling

```mermaid
graph LR
    subgraph "User Data"
        MESSAGES[Messages]
        METADATA[Metadata]
        PII[Personal Info]
        MEDIA_PII[Media Files]
    end
    
    subgraph "Privacy Controls"
        ANONYMIZE[Anonymization]
        REDACT[PII Redaction]
        MINIMIZE[Data Minimization]
        RETENTION_PII[Retention Policies]
    end
    
    subgraph "Compliance"
        GDPR[GDPR Compliance]
        CCPA[CCPA Compliance]
        AUDIT_PII[Audit Trail]
        CONSENT[User Consent]
    end
    
    subgraph "User Rights"
        ACCESS[Data Access]
        EXPORT[Data Export]
        DELETE[Data Deletion]
        PORTABILITY[Data Portability]
    end
    
    MESSAGES --> ANONYMIZE
    METADATA --> REDACT
    PII --> MINIMIZE
    MEDIA_PII --> RETENTION_PII
    
    ANONYMIZE --> GDPR
    REDACT --> CCPA
    MINIMIZE --> AUDIT_PII
    RETENTION_PII --> CONSENT
    
    GDPR --> ACCESS
    CCPA --> EXPORT
    AUDIT_PII --> DELETE
    CONSENT --> PORTABILITY
    
    style ANONYMIZE fill:#4ecdc4
    style GDPR fill:#ff6b6b
    style CONSENT fill:#51cf66
```

## Audit & Compliance

```mermaid
graph TB
    subgraph "Audit Events"
        AUTH_EVENT[Authentication]
        ACCESS_EVENT[Data Access]
        CONFIG_EVENT[Config Changes]
        SECURITY_EVENT[Security Events]
    end
    
    subgraph "Logging"
        STRUCTURED_LOG[Structured Logs]
        TIMESTAMP[Timestamps]
        CORRELATION[Correlation IDs]
        IMMUTABLE[Immutable Records]
    end
    
    subgraph "Storage"
        LOCAL_LOGS[Local Storage]
        ENCRYPTED_LOGS[Encrypted Logs]
        RETENTION_LOGS[Retention Period]
        ARCHIVE_LOGS[Archival]
    end
    
    subgraph "Analysis"
        SEARCH_LOGS[Log Search]
        REPORT[Compliance Reports]
        FORENSICS[Forensic Analysis]
        EXPORT_LOGS[Export for Audit]
    end
    
    AUTH_EVENT --> STRUCTURED_LOG
    ACCESS_EVENT --> STRUCTURED_LOG
    CONFIG_EVENT --> STRUCTURED_LOG
    SECURITY_EVENT --> STRUCTURED_LOG
    
    STRUCTURED_LOG --> TIMESTAMP
    TIMESTAMP --> CORRELATION
    CORRELATION --> IMMUTABLE
    
    IMMUTABLE --> LOCAL_LOGS
    LOCAL_LOGS --> ENCRYPTED_LOGS
    ENCRYPTED_LOGS --> RETENTION_LOGS
    RETENTION_LOGS --> ARCHIVE_LOGS
    
    ARCHIVE_LOGS --> SEARCH_LOGS
    SEARCH_LOGS --> REPORT
    REPORT --> FORENSICS
    FORENSICS --> EXPORT_LOGS
    
    style STRUCTURED_LOG fill:#4ecdc4
    style IMMUTABLE fill:#ff6b6b
    style ENCRYPTED_LOGS fill:#95e1d3
```
