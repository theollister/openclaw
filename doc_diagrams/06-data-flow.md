# OpenClaw Data Flow Architecture

## Configuration Data Flow

```mermaid
graph TB
    subgraph "Configuration Sources"
        ENV[Environment<br/>Variables]
        FILE[Config File<br/>~/.openclaw/config.json]
        CLI[CLI Arguments<br/>--flag value]
        DEFAULT[Default Values]
    end
    
    subgraph "Config System"
        LOADER[Config Loader]
        MERGE[Merge Strategy]
        VALIDATE[Validation]
        SCHEMA[JSON Schema]
    end
    
    subgraph "Runtime Access"
        GATEWAY[Gateway]
        PLUGINS[Plugins]
        CHANNELS[Channels]
        PROVIDERS[Providers]
    end
    
    subgraph "Storage"
        PERSIST[Persist Changes]
        WATCH[File Watcher]
        RELOAD[Hot Reload]
    end
    
    DEFAULT --> LOADER
    FILE --> LOADER
    ENV --> LOADER
    CLI --> LOADER
    
    LOADER --> MERGE
    MERGE --> VALIDATE
    VALIDATE --> SCHEMA
    
    SCHEMA --> GATEWAY
    SCHEMA --> PLUGINS
    SCHEMA --> CHANNELS
    SCHEMA --> PROVIDERS
    
    GATEWAY --> PERSIST
    PERSIST --> FILE
    FILE --> WATCH
    WATCH --> RELOAD
    RELOAD --> GATEWAY
    
    style LOADER fill:#4ecdc4
    style MERGE fill:#95e1d3
    style GATEWAY fill:#ff6b6b
```

## Credential Management

```mermaid
graph LR
    subgraph "Credential Storage"
        OAUTH[OAuth Tokens<br/>~/.openclaw/credentials/]
        APIKEYS[API Keys<br/>~/.openclaw/credentials/]
        SECRETS[Secrets<br/>Encrypted]
    end
    
    subgraph "Provider Authentication"
        ANTHROPIC[Anthropic]
        OPENAI[OpenAI]
        TOGETHER[Together.ai]
        FIREWORKS[Fireworks]
        OLLAMA[Ollama Local]
    end
    
    subgraph "OAuth Flow"
        LOGIN[openclaw login]
        BROWSER[Browser Opens]
        CALLBACK[OAuth Callback]
        TOKEN[Store Token]
    end
    
    subgraph "Security"
        ENCRYPT[Encryption]
        PERMS[File Permissions]
        ROTATE[Token Rotation]
    end
    
    OAUTH --> ANTHROPIC
    OAUTH --> OPENAI
    APIKEYS --> TOGETHER
    APIKEYS --> FIREWORKS
    
    LOGIN --> BROWSER
    BROWSER --> CALLBACK
    CALLBACK --> TOKEN
    TOKEN --> OAUTH
    
    OAUTH --> ENCRYPT
    APIKEYS --> ENCRYPT
    SECRETS --> ENCRYPT
    
    ENCRYPT --> PERMS
    ENCRYPT --> ROTATE
    
    style OAUTH fill:#95e1d3
    style ENCRYPT fill:#ff6b6b
```

## Session & Memory Storage

```mermaid
graph TB
    subgraph "Session Storage"
        SESSIONS[Sessions<br/>~/.openclaw/sessions/]
        AGENTS[Agent Sessions<br/>~/.openclaw/agents/]
        LOGS[Session Logs<br/>*.jsonl]
    end
    
    subgraph "Memory System"
        STM[Short-term Memory<br/>In-process cache]
        LTM[Long-term Memory<br/>Persistent storage]
        VECTOR[Vector Store<br/>LanceDB/etc]
    end
    
    subgraph "Context Management"
        CONTEXT[Conversation Context]
        HISTORY[Message History]
        EMBEDDINGS[Semantic Search]
    end
    
    subgraph "Cleanup"
        PRUNE[Prune Old Sessions]
        ARCHIVE[Archive Logs]
        RETENTION[Retention Policy]
    end
    
    SESSIONS --> STM
    AGENTS --> STM
    LOGS --> STM
    
    STM --> CONTEXT
    LTM --> CONTEXT
    VECTOR --> CONTEXT
    
    CONTEXT --> HISTORY
    CONTEXT --> EMBEDDINGS
    
    SESSIONS --> PRUNE
    LOGS --> ARCHIVE
    PRUNE --> RETENTION
    ARCHIVE --> RETENTION
    
    style STM fill:#4ecdc4
    style LTM fill:#95e1d3
    style VECTOR fill:#ffd93d
```

## Media Pipeline Data Flow

```mermaid
sequenceDiagram
    participant User
    participant Channel
    participant Gateway
    participant Media as Media Pipeline
    participant Storage
    participant AI as AI Provider
    
    User->>Channel: Send image/video/audio
    Channel->>Gateway: Forward media
    Gateway->>Media: Process media
    
    alt Image Processing
        Media->>Media: Decode image
        Media->>Media: Resize/optimize
        Media->>Storage: Store original
        Media->>AI: Send for vision analysis
        AI-->>Media: Return analysis
    else Video Processing
        Media->>Media: Extract frames
        Media->>Media: Generate thumbnail
        Media->>Storage: Store video
        Media->>AI: Send frames for analysis
        AI-->>Media: Return analysis
    else Audio Processing
        Media->>Media: Transcribe audio
        Media->>Storage: Store audio
        Media->>AI: Send for analysis
        AI-->>Media: Return transcription
    end
    
    Media-->>Gateway: Return processed media
    Gateway-->>Channel: Send response
    Channel-->>User: Display result
```

## Logging & Observability

```mermaid
graph TB
    subgraph "Log Sources"
        GATEWAY_LOG[Gateway Logs]
        CHANNEL_LOG[Channel Logs]
        PLUGIN_LOG[Plugin Logs]
        ERROR_LOG[Error Logs]
    end
    
    subgraph "Log Processing"
        STRUCT[Structured Logging]
        FORMAT[Log Formatter]
        LEVEL[Log Levels]
    end
    
    subgraph "Log Destinations"
        CONSOLE[Console Output]
        FILE_LOG[Log Files]
        SYSLOG[System Logs<br/>macOS unified log]
        OTEL[OpenTelemetry]
    end
    
    subgraph "Monitoring"
        METRICS[Metrics]
        TRACES[Distributed Tracing]
        ALERTS[Alerting]
    end
    
    GATEWAY_LOG --> STRUCT
    CHANNEL_LOG --> STRUCT
    PLUGIN_LOG --> STRUCT
    ERROR_LOG --> STRUCT
    
    STRUCT --> FORMAT
    FORMAT --> LEVEL
    
    LEVEL --> CONSOLE
    LEVEL --> FILE_LOG
    LEVEL --> SYSLOG
    LEVEL --> OTEL
    
    OTEL --> METRICS
    OTEL --> TRACES
    METRICS --> ALERTS
    
    style STRUCT fill:#4ecdc4
    style OTEL fill:#95e1d3
    style METRICS fill:#ffd93d
```

## Pairing & Device Sync

```mermaid
graph LR
    subgraph "Primary Device"
        MAIN[Main Gateway]
        CONFIG_MAIN[Config]
        CREDS_MAIN[Credentials]
    end
    
    subgraph "Pairing Process"
        CODE[Pairing Code]
        QR[QR Code]
        VERIFY[Verification]
    end
    
    subgraph "Secondary Device"
        SECONDARY[Secondary Gateway]
        CONFIG_SEC[Config]
        CREDS_SEC[Credentials]
    end
    
    subgraph "Sync Data"
        CHANNELS_SYNC[Channel Config]
        ALLOWLIST_SYNC[Allowlists]
        ROUTING_SYNC[Routing Rules]
    end
    
    MAIN --> CODE
    CODE --> QR
    QR --> SECONDARY
    
    SECONDARY --> VERIFY
    VERIFY --> MAIN
    
    MAIN --> CHANNELS_SYNC
    MAIN --> ALLOWLIST_SYNC
    MAIN --> ROUTING_SYNC
    
    CHANNELS_SYNC --> SECONDARY
    ALLOWLIST_SYNC --> SECONDARY
    ROUTING_SYNC --> SECONDARY
    
    CONFIG_MAIN -.Sync.-> CONFIG_SEC
    CREDS_MAIN -.Secure Sync.-> CREDS_SEC
    
    style CODE fill:#4ecdc4
    style VERIFY fill:#51cf66
    style CHANNELS_SYNC fill:#95e1d3
```

## Cache & Performance

```mermaid
graph TB
    subgraph "Cache Layers"
        L1[L1: In-Memory<br/>Hot data]
        L2[L2: Redis/Local<br/>Warm data]
        L3[L3: Disk<br/>Cold data]
    end
    
    subgraph "Cached Data"
        PROVIDERS[Provider Responses]
        MEDIA_CACHE[Media Assets]
        CONTEXT_CACHE[Conversation Context]
        CONFIG_CACHE[Config Values]
    end
    
    subgraph "Cache Strategy"
        TTL[TTL Expiration]
        LRU[LRU Eviction]
        INVALIDATE[Invalidation]
    end
    
    subgraph "Performance"
        METRICS_PERF[Response Times]
        HITRATE[Cache Hit Rate]
        OPTIMIZE[Optimization]
    end
    
    PROVIDERS --> L1
    MEDIA_CACHE --> L2
    CONTEXT_CACHE --> L1
    CONFIG_CACHE --> L1
    
    L1 --> TTL
    L2 --> LRU
    L3 --> INVALIDATE
    
    L1 --> METRICS_PERF
    L2 --> HITRATE
    L3 --> OPTIMIZE
    
    style L1 fill:#51cf66
    style L2 fill:#4ecdc4
    style L3 fill:#95e1d3
```
