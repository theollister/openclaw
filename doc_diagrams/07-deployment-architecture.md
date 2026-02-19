# OpenClaw Deployment Architecture

## Deployment Scenarios

```mermaid
graph TB
    subgraph "Local Deployment"
        LOCAL_MAC[macOS Menubar App]
        LOCAL_PI[Raspberry Pi]
        LOCAL_LINUX[Linux Server]
        LOCAL_WIN[Windows WSL2]
    end
    
    subgraph "Cloud Deployment"
        FLY[Fly.io]
        DOCKER[Docker Container]
        VPS[VPS Server]
        K8S[Kubernetes]
    end
    
    subgraph "Hybrid Deployment"
        GATEWAY_CLOUD[Cloud Gateway]
        GATEWAY_LOCAL[Local Gateway]
        TUNNEL[Tailscale/SSH Tunnel]
    end
    
    subgraph "Services"
        CHANNELS[Channel Handlers]
        AI[AI Providers]
        STORAGE[Storage]
        LOGS[Logging]
    end
    
    LOCAL_MAC --> CHANNELS
    LOCAL_PI --> CHANNELS
    LOCAL_LINUX --> CHANNELS
    LOCAL_WIN --> CHANNELS
    
    FLY --> CHANNELS
    DOCKER --> CHANNELS
    VPS --> CHANNELS
    K8S --> CHANNELS
    
    GATEWAY_CLOUD --> TUNNEL
    TUNNEL --> GATEWAY_LOCAL
    GATEWAY_CLOUD --> CHANNELS
    GATEWAY_LOCAL --> CHANNELS
    
    CHANNELS --> AI
    CHANNELS --> STORAGE
    CHANNELS --> LOGS
    
    style LOCAL_MAC fill:#4ecdc4
    style FLY fill:#ff6b6b
    style TUNNEL fill:#95e1d3
```

## macOS Deployment

```mermaid
graph LR
    subgraph "macOS App"
        MENUBAR[Menubar Icon]
        UI[SwiftUI Interface]
        GATEWAY_MAC[Gateway Process]
        LAUNCHD[LaunchAgent]
    end
    
    subgraph "Gateway Services"
        SERVER[HTTP Server]
        WEBSOCKET[WebSocket Server]
        CHANNELS_MAC[Channel Handlers]
    end
    
    subgraph "System Integration"
        KEYCHAIN[Keychain Access]
        UNIFIEDLOG[Unified Logging]
        NOTIFICATIONS[Notifications]
        VOICEWAKE[Voice Wake Word]
    end
    
    MENUBAR --> UI
    UI --> GATEWAY_MAC
    GATEWAY_MAC --> LAUNCHD
    LAUNCHD --> GATEWAY_MAC
    
    GATEWAY_MAC --> SERVER
    GATEWAY_MAC --> WEBSOCKET
    GATEWAY_MAC --> CHANNELS_MAC
    
    GATEWAY_MAC --> KEYCHAIN
    GATEWAY_MAC --> UNIFIEDLOG
    GATEWAY_MAC --> NOTIFICATIONS
    GATEWAY_MAC --> VOICEWAKE
    
    style MENUBAR fill:#4ecdc4
    style GATEWAY_MAC fill:#ff6b6b
```

## Raspberry Pi Deployment

```mermaid
graph TB
    subgraph "Pi Setup"
        PI[Raspberry Pi 4/5]
        OS[Raspberry Pi OS]
        NODE[Node.js 22+]
        SYSTEMD[systemd Service]
    end
    
    subgraph "Gateway Install"
        NPM[npm install -g openclaw]
        ONBOARD[openclaw onboard]
        DAEMON[Install Daemon]
        CONFIG[Configuration]
    end
    
    subgraph "Run Mode"
        SERVICE[systemd User Service]
        GATEWAY_PI[Gateway Process]
        AUTOSTART[Auto-start on boot]
    end
    
    subgraph "Storage"
        CONFIG_DIR[~/.openclaw/]
        SESSIONS_DIR[~/.openclaw/sessions/]
        CREDS[~/.openclaw/credentials/]
        LOGS_DIR[Log files]
    end
    
    PI --> OS
    OS --> NODE
    NODE --> NPM
    NPM --> ONBOARD
    ONBOARD --> DAEMON
    DAEMON --> CONFIG
    
    DAEMON --> SYSTEMD
    SYSTEMD --> SERVICE
    SERVICE --> GATEWAY_PI
    SERVICE --> AUTOSTART
    
    GATEWAY_PI --> CONFIG_DIR
    GATEWAY_PI --> SESSIONS_DIR
    GATEWAY_PI --> CREDS
    GATEWAY_PI --> LOGS_DIR
    
    style PI fill:#4ecdc4
    style GATEWAY_PI fill:#ff6b6b
    style SERVICE fill:#95e1d3
```

## Docker Deployment

```mermaid
graph TB
    subgraph "Docker Image"
        DOCKERFILE[Dockerfile]
        BASE[Node.js Base]
        DEPS[Dependencies]
        BUILD[Build Artifacts]
    end
    
    subgraph "Container Runtime"
        CONTAINER[Docker Container]
        VOLUMES[Volume Mounts]
        NETWORK[Network Config]
        ENV[Environment Variables]
    end
    
    subgraph "Compose Stack"
        GATEWAY_DOCKER[openclaw-gateway]
        REDIS[Redis Cache]
        LANCEDB[LanceDB]
        NGINX[Nginx Proxy]
    end
    
    subgraph "Orchestration"
        COMPOSE[docker-compose.yml]
        SCALE[Scaling]
        HEALTH[Health Checks]
    end
    
    DOCKERFILE --> BASE
    BASE --> DEPS
    DEPS --> BUILD
    
    BUILD --> CONTAINER
    CONTAINER --> VOLUMES
    CONTAINER --> NETWORK
    CONTAINER --> ENV
    
    CONTAINER --> GATEWAY_DOCKER
    COMPOSE --> REDIS
    COMPOSE --> LANCEDB
    COMPOSE --> NGINX
    
    COMPOSE --> SCALE
    COMPOSE --> HEALTH
    
    GATEWAY_DOCKER --> HEALTH
    
    style DOCKERFILE fill:#4ecdc4
    style CONTAINER fill:#ff6b6b
    style COMPOSE fill:#95e1d3
```

## Cloud VPS Deployment

```mermaid
sequenceDiagram
    participant Dev as Developer
    participant VPS as VPS Server
    participant Systemd
    participant Gateway
    participant Channels
    
    Dev->>VPS: SSH connect
    VPS->>VPS: Install Node.js 22+
    VPS->>VPS: npm install -g openclaw
    
    Dev->>VPS: openclaw onboard
    VPS->>VPS: Configure channels
    VPS->>VPS: Configure AI providers
    VPS->>VPS: Set allowlists
    
    Dev->>VPS: openclaw onboard --install-daemon
    VPS->>Systemd: Install user service
    Systemd->>Gateway: Start gateway
    Gateway->>Channels: Connect channels
    
    loop Health Check
        Gateway->>Gateway: Self monitor
        Gateway->>Systemd: Report status
    end
    
    alt Gateway Crash
        Systemd->>Gateway: Auto-restart
        Gateway->>Channels: Reconnect
    end
```

## Fly.io Deployment

```mermaid
graph LR
    subgraph "Fly Configuration"
        FLY_TOML[fly.toml]
        DOCKERFILE_FLY[Dockerfile]
        SECRETS[Fly Secrets]
    end
    
    subgraph "Fly Platform"
        MACHINE[Fly Machine]
        VOLUME[Persistent Volume]
        NETWORK_FLY[Private Network]
        PROXY[Fly Proxy]
    end
    
    subgraph "Gateway Runtime"
        APP[OpenClaw Gateway]
        DATA[Data Directory]
        LOGS_FLY[Logs]
    end
    
    subgraph "Management"
        DEPLOY[fly deploy]
        RESTART[fly machines restart]
        LOGS_CMD[fly logs]
        SSH_CMD[fly ssh console]
    end
    
    FLY_TOML --> MACHINE
    DOCKERFILE_FLY --> MACHINE
    SECRETS --> MACHINE
    
    MACHINE --> VOLUME
    MACHINE --> NETWORK_FLY
    MACHINE --> PROXY
    
    MACHINE --> APP
    APP --> DATA
    APP --> LOGS_FLY
    
    DEPLOY --> MACHINE
    RESTART --> MACHINE
    LOGS_CMD --> LOGS_FLY
    SSH_CMD --> MACHINE
    
    style FLY_TOML fill:#4ecdc4
    style MACHINE fill:#ff6b6b
    style APP fill:#95e1d3
```

## Network Architecture

```mermaid
graph TB
    subgraph "External"
        USER[Users]
        CHANNELS_EXT[Channel APIs<br/>Telegram/Discord/etc]
        AI_EXT[AI Provider APIs<br/>Anthropic/OpenAI]
    end
    
    subgraph "Edge Layer"
        FIREWALL[Firewall]
        TAILSCALE[Tailscale VPN]
        REVERSE_PROXY[Reverse Proxy]
    end
    
    subgraph "Gateway Layer"
        GATEWAY_NET[Gateway Server]
        WEB_UI[Web Interface]
        API[REST API]
        WS[WebSocket]
    end
    
    subgraph "Internal Services"
        CHANNELS_INT[Channel Handlers]
        PROVIDERS[Provider Clients]
        STORAGE_NET[Storage Layer]
    end
    
    USER --> CHANNELS_EXT
    CHANNELS_EXT --> FIREWALL
    
    FIREWALL --> TAILSCALE
    FIREWALL --> REVERSE_PROXY
    TAILSCALE --> GATEWAY_NET
    REVERSE_PROXY --> GATEWAY_NET
    
    GATEWAY_NET --> WEB_UI
    GATEWAY_NET --> API
    GATEWAY_NET --> WS
    
    GATEWAY_NET --> CHANNELS_INT
    GATEWAY_NET --> PROVIDERS
    GATEWAY_NET --> STORAGE_NET
    
    CHANNELS_INT --> CHANNELS_EXT
    PROVIDERS --> AI_EXT
    
    style FIREWALL fill:#ff6b6b
    style GATEWAY_NET fill:#4ecdc4
    style TAILSCALE fill:#95e1d3
```

## High Availability Setup

```mermaid
graph TB
    subgraph "Load Balancer"
        LB[Load Balancer]
        HEALTH_LB[Health Checks]
    end
    
    subgraph "Gateway Cluster"
        GW1[Gateway Instance 1]
        GW2[Gateway Instance 2]
        GW3[Gateway Instance 3]
    end
    
    subgraph "Shared Services"
        REDIS_CLUSTER[Redis Cluster]
        DB[Shared Database]
        STORAGE_SHARED[Shared Storage]
    end
    
    subgraph "Monitoring"
        METRICS_CLUSTER[Metrics]
        LOGS_CLUSTER[Centralized Logs]
        ALERTS_CLUSTER[Alerting]
    end
    
    LB --> HEALTH_LB
    HEALTH_LB --> GW1
    HEALTH_LB --> GW2
    HEALTH_LB --> GW3
    
    GW1 --> REDIS_CLUSTER
    GW2 --> REDIS_CLUSTER
    GW3 --> REDIS_CLUSTER
    
    GW1 --> DB
    GW2 --> DB
    GW3 --> DB
    
    GW1 --> STORAGE_SHARED
    GW2 --> STORAGE_SHARED
    GW3 --> STORAGE_SHARED
    
    GW1 --> METRICS_CLUSTER
    GW2 --> METRICS_CLUSTER
    GW3 --> METRICS_CLUSTER
    
    METRICS_CLUSTER --> LOGS_CLUSTER
    LOGS_CLUSTER --> ALERTS_CLUSTER
    
    style LB fill:#4ecdc4
    style REDIS_CLUSTER fill:#ff6b6b
    style METRICS_CLUSTER fill:#ffd93d
```
