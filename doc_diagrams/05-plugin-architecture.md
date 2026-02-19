# OpenClaw Plugin Architecture

## Plugin System Overview

```mermaid
graph TB
    subgraph "Plugin SDK"
        SDK[Plugin SDK]
        API[Extension API]
        TYPES[TypeScript Types]
        LOADER[Plugin Loader]
    end
    
    subgraph "Core Services"
        GATEWAY[Gateway]
        ROUTER[Router]
        HOOKS[Hooks]
        CONFIG[Config]
    end
    
    subgraph "Plugin Types"
        CHANNEL[Channel Plugins]
        AUTH[Auth Plugins]
        MEMORY[Memory Plugins]
        TOOL[Tool Plugins]
        FEATURE[Feature Plugins]
    end
    
    subgraph "Plugin Lifecycle"
        DISCOVER[Discover]
        LOAD[Load]
        INIT[Initialize]
        REGISTER[Register]
        RUN[Run]
    end
    
    SDK --> API
    SDK --> TYPES
    SDK --> LOADER
    
    API --> GATEWAY
    API --> ROUTER
    API --> HOOKS
    API --> CONFIG
    
    LOADER --> CHANNEL
    LOADER --> AUTH
    LOADER --> MEMORY
    LOADER --> TOOL
    LOADER --> FEATURE
    
    DISCOVER --> LOAD
    LOAD --> INIT
    INIT --> REGISTER
    REGISTER --> RUN
    
    style SDK fill:#95e1d3
    style LOADER fill:#4ecdc4
    style GATEWAY fill:#ff6b6b
```

## Plugin Discovery & Loading

```mermaid
sequenceDiagram
    participant Gateway
    participant Loader as Plugin Loader
    participant FS as File System
    participant Plugin
    participant Registry
    
    Gateway->>Loader: Initialize plugin system
    Loader->>FS: Scan extensions/ directory
    FS-->>Loader: Return plugin packages
    
    loop For each plugin
        Loader->>FS: Read package.json
        FS-->>Loader: Plugin metadata
        
        Loader->>Loader: Validate plugin schema
        
        alt Valid plugin
            Loader->>Plugin: Load plugin module
            Plugin-->>Loader: Export plugin interface
            
            Loader->>Plugin: Call initialize()
            Plugin-->>Loader: Return capabilities
            
            Loader->>Registry: Register plugin
            Registry-->>Loader: Registration complete
        else Invalid plugin
            Loader->>Loader: Log error & skip
        end
    end
    
    Loader-->>Gateway: All plugins loaded
```

## Extension Package Structure

```mermaid
graph LR
    subgraph "Extension Directory"
        ROOT[extensions/example/]
        PKG[package.json]
        SRC[src/]
        INDEX[src/index.ts]
        HANDLER[src/handler.ts]
        CONFIG[src/config.ts]
        README[README.md]
    end
    
    subgraph "package.json"
        NAME[name: openclaw-example]
        DEPS[dependencies]
        DEVDEPS[devDependencies]
        MAIN[main: dist/index.js]
    end
    
    subgraph "Plugin Interface"
        EXPORT[export default plugin]
        INIT[initialize function]
        CHANNELS[channel handlers]
        HOOKS[hook registrations]
    end
    
    ROOT --> PKG
    ROOT --> SRC
    ROOT --> README
    
    SRC --> INDEX
    SRC --> HANDLER
    SRC --> CONFIG
    
    PKG --> NAME
    PKG --> DEPS
    PKG --> DEVDEPS
    PKG --> MAIN
    
    INDEX --> EXPORT
    EXPORT --> INIT
    EXPORT --> CHANNELS
    EXPORT --> HOOKS
    
    style ROOT fill:#95e1d3
    style PKG fill:#4ecdc4
    style EXPORT fill:#ff6b6b
```

## Plugin Types & Use Cases

### Channel Plugins
Add support for new messaging platforms:
- `extensions/discord/` - Discord integration
- `extensions/signal/` - Signal messaging
- `extensions/matrix/` - Matrix protocol
- `extensions/msteams/` - Microsoft Teams
- `extensions/zalo/` - Zalo messenger

### Authentication Plugins
Custom OAuth/Auth flows:
- `extensions/google-antigravity-auth/` - Google AI auth
- `extensions/google-gemini-cli-auth/` - Gemini CLI auth
- `extensions/minimax-portal-auth/` - MiniMax portal
- `extensions/qwen-portal-auth/` - Qwen portal

### Memory Plugins
Custom memory backends:
- `extensions/memory-core/` - Core memory interface
- `extensions/memory-lancedb/` - LanceDB vector store

### Feature Plugins
Extended capabilities:
- `extensions/phone-control/` - Phone automation
- `extensions/voice-call/` - Voice calling
- `extensions/talk-voice/` - Voice synthesis
- `extensions/device-pair/` - Device pairing
- `extensions/diagnostics-otel/` - OpenTelemetry

### Tool Plugins
AI tool integrations:
- `extensions/llm-task/` - LLM task delegation
- `extensions/open-prose/` - Prose generation
- `extensions/lobster/` - Lobster color palette

## Plugin API Surface

```mermaid
classDiagram
    class PluginSDK {
        +registerChannel()
        +registerHook()
        +registerTool()
        +getConfig()
        +getLogger()
        +sendMessage()
        +receiveMessage()
    }
    
    class ChannelPlugin {
        +name: string
        +initialize()
        +sendMessage()
        +receiveMessage()
        +status()
        +disconnect()
    }
    
    class HookPlugin {
        +name: string
        +trigger: string
        +priority: number
        +execute()
    }
    
    class ToolPlugin {
        +name: string
        +description: string
        +parameters: Schema
        +execute()
    }
    
    class MemoryPlugin {
        +name: string
        +initialize()
        +store()
        +retrieve()
        +search()
    }
    
    PluginSDK <|-- ChannelPlugin
    PluginSDK <|-- HookPlugin
    PluginSDK <|-- ToolPlugin
    PluginSDK <|-- MemoryPlugin
```

## Plugin Configuration

```mermaid
graph LR
    subgraph "Configuration Sources"
        ENV[Environment Variables]
        FILE[Config File]
        CLI[CLI Arguments]
        DEFAULT[Plugin Defaults]
    end
    
    subgraph "Config Merge"
        MERGE[Config Merger]
        VALIDATE[Validation]
        SCHEMA[JSON Schema]
    end
    
    subgraph "Plugin Access"
        PLUGIN[Plugin Code]
        ACCESS[config.get key]
        WATCH[config.watch key]
    end
    
    ENV --> MERGE
    FILE --> MERGE
    CLI --> MERGE
    DEFAULT --> MERGE
    
    MERGE --> VALIDATE
    VALIDATE --> SCHEMA
    SCHEMA --> PLUGIN
    
    PLUGIN --> ACCESS
    PLUGIN --> WATCH
    
    style MERGE fill:#4ecdc4
    style PLUGIN fill:#95e1d3
```

## Plugin Installation & Runtime

### Installation Flow
```bash
# Plugin dependencies installed separately
npm install --omit=dev  # In plugin directory

# Runtime dependencies in 'dependencies'
# Dev dependencies in 'devDependencies'
# Never use 'workspace:*' in dependencies
```

### Runtime Resolution
- Plugins loaded via jiti (just-in-time TypeScript)
- `openclaw/plugin-sdk` resolved via jiti alias
- Core dependencies resolved from parent node_modules
- Plugin dependencies isolated in plugin's node_modules

### Dependency Guidelines
```json
{
  "dependencies": {
    "third-party-lib": "^1.0.0"
  },
  "devDependencies": {
    "openclaw": "workspace:*"
  },
  "peerDependencies": {
    "openclaw": "*"
  }
}
```

## Plugin Security & Isolation

```mermaid
graph TB
    subgraph "Security Layers"
        SANDBOX[Sandbox Environment]
        PERMS[Permission System]
        AUDIT[Audit Logging]
    end
    
    subgraph "Plugin Runtime"
        PLUGIN[Plugin Code]
        API[Limited API Surface]
        RESTRICT[Restricted Access]
    end
    
    subgraph "Protected Resources"
        CREDS[Credentials]
        CONFIG[System Config]
        FILES[File System]
        NETWORK[Network]
    end
    
    PLUGIN --> API
    API --> PERMS
    PERMS --> SANDBOX
    SANDBOX --> AUDIT
    
    RESTRICT --> CREDS
    RESTRICT --> CONFIG
    RESTRICT --> FILES
    RESTRICT --> NETWORK
    
    PERMS -.Check.-> RESTRICT
    
    style SANDBOX fill:#51cf66
    style RESTRICT fill:#ff6b6b
    style API fill:#4ecdc4
```
