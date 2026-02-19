# OpenClaw Message Flow Architecture

## Message Routing Flow

```mermaid
sequenceDiagram
    participant User
    participant Channel as Channel<br/>(Telegram/Discord/etc)
    participant Gateway as Gateway Server
    participant Router as Message Router
    participant Hooks as Hooks System
    participant Agent as AI Agent
    participant Provider as AI Provider<br/>(Claude/GPT)
    participant Memory as Memory System
    
    User->>Channel: Send message
    Channel->>Gateway: Webhook/WebSocket
    Gateway->>Router: Route incoming message
    
    Router->>Router: Check allowlists
    Router->>Router: Apply routing rules
    
    alt Message allowed
        Router->>Hooks: Trigger pre-process hooks
        Hooks-->>Router: Transform/validate message
        
        Router->>Agent: Process message
        
        Agent->>Memory: Load conversation context
        Memory-->>Agent: Return history
        
        Agent->>Provider: Send prompt + context
        Provider-->>Agent: Stream response
        
        Agent->>Hooks: Trigger post-process hooks
        Hooks-->>Agent: Transform response
        
        Agent->>Router: Return response
        Router->>Gateway: Send to channel
        Gateway->>Channel: Deliver message
        Channel->>User: Display response
        
        Router->>Memory: Store conversation
    else Message blocked
        Router->>Gateway: Log + drop
        Gateway->>Channel: Optional error response
    end
```

## Channel Registration & Discovery

```mermaid
graph TB
    subgraph "Channel Types"
        CORE[Core Channels]
        EXT[Extension Channels]
    end
    
    subgraph "Registration Process"
        DISCOVER[Channel Discovery]
        VALIDATE[Validate Config]
        REGISTER[Register Handler]
        STATUS[Status Check]
    end
    
    subgraph "Channel Lifecycle"
        INIT[Initialize]
        CONNECT[Connect]
        LISTEN[Listen]
        SEND[Send]
        DISCONNECT[Disconnect]
    end
    
    CORE --> DISCOVER
    EXT --> DISCOVER
    
    DISCOVER --> VALIDATE
    VALIDATE --> REGISTER
    REGISTER --> STATUS
    
    STATUS --> INIT
    INIT --> CONNECT
    CONNECT --> LISTEN
    LISTEN --> SEND
    SEND --> LISTEN
    LISTEN --> DISCONNECT
    
    style CORE fill:#ff6b6b
    style EXT fill:#95e1d3
    style REGISTER fill:#4ecdc4
```

## Routing Rules & Allowlists

```mermaid
graph LR
    subgraph "Incoming Message"
        MSG[Message]
        SENDER[Sender ID]
        CHANNEL[Channel Info]
        CONTENT[Content]
    end
    
    subgraph "Routing Checks"
        CHECK1[Allowlist Check]
        CHECK2[Pairing Check]
        CHECK3[Rate Limit]
        CHECK4[Command Gating]
    end
    
    subgraph "Routing Decisions"
        ALLOW[Process Message]
        DENY[Block/Log]
        QUEUE[Queue for Later]
    end
    
    MSG --> CHECK1
    SENDER --> CHECK1
    CHANNEL --> CHECK1
    CONTENT --> CHECK1
    
    CHECK1 -->|Allowed| CHECK2
    CHECK1 -->|Blocked| DENY
    
    CHECK2 -->|Paired| CHECK3
    CHECK2 -->|Unpaired| DENY
    
    CHECK3 -->|Under Limit| CHECK4
    CHECK3 -->|Rate Limited| QUEUE
    
    CHECK4 -->|Authorized| ALLOW
    CHECK4 -->|Unauthorized| DENY
    
    style ALLOW fill:#51cf66
    style DENY fill:#ff6b6b
    style QUEUE fill:#ffd93d
```

## Multi-Channel Support

All built-in and extension channels follow the same routing architecture:

### Built-in Channels (Core)
- Telegram (`src/telegram`)
- Discord (`src/discord`)
- Slack (`src/slack`)
- Signal (`src/signal`)
- iMessage (`src/imessage`)
- WhatsApp Web (`src/web`)
- Google Chat (via extension)
- Microsoft Teams (via extension)

### Extension Channels
- BlueBubbles (`extensions/bluebubbles`)
- Matrix (`extensions/matrix`)
- Zalo (`extensions/zalo`)
- Zalo Personal (`extensions/zalouser`)
- Feishu (`extensions/feishu`)
- Mattermost (`extensions/mattermost`)
- Nextcloud Talk (`extensions/nextcloud-talk`)
- Voice Call (`extensions/voice-call`)

Each channel implements:
1. **Message Handler**: Receives and parses incoming messages
2. **Sender**: Formats and sends outgoing messages
3. **Status Check**: Health and connectivity validation
4. **Configuration**: Channel-specific settings and credentials
