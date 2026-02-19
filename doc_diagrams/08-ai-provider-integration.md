# OpenClaw AI Provider Integration

## Multi-Provider Architecture

```mermaid
graph TB
    subgraph "Agent Layer"
        AGENT[AI Agent]
        ROUTER_AI[Provider Router]
        FAILOVER[Failover Logic]
        CACHE_AI[Response Cache]
    end
    
    subgraph "Provider Adapters"
        ANTHROPIC_ADAPTER[Anthropic Adapter]
        OPENAI_ADAPTER[OpenAI Adapter]
        OLLAMA_ADAPTER[Ollama Adapter]
        TOGETHER_ADAPTER[Together Adapter]
        FIREWORKS_ADAPTER[Fireworks Adapter]
        CUSTOM_ADAPTER[Custom Adapter]
    end
    
    subgraph "External APIs"
        ANTHROPIC_API[Anthropic API<br/>Claude 4.6 Opus/Sonnet]
        OPENAI_API[OpenAI API<br/>GPT-4/o1]
        OLLAMA_API[Ollama Local<br/>Llama/Mistral]
        TOGETHER_API[Together.ai API]
        FIREWORKS_API[Fireworks API]
    end
    
    subgraph "Authentication"
        OAUTH[OAuth Tokens]
        APIKEY[API Keys]
        LOCAL[Local Auth]
    end
    
    AGENT --> ROUTER_AI
    ROUTER_AI --> FAILOVER
    ROUTER_AI --> CACHE_AI
    
    ROUTER_AI --> ANTHROPIC_ADAPTER
    ROUTER_AI --> OPENAI_ADAPTER
    ROUTER_AI --> OLLAMA_ADAPTER
    ROUTER_AI --> TOGETHER_ADAPTER
    ROUTER_AI --> FIREWORKS_ADAPTER
    ROUTER_AI --> CUSTOM_ADAPTER
    
    ANTHROPIC_ADAPTER --> ANTHROPIC_API
    OPENAI_ADAPTER --> OPENAI_API
    OLLAMA_ADAPTER --> OLLAMA_API
    TOGETHER_ADAPTER --> TOGETHER_API
    FIREWORKS_ADAPTER --> FIREWORKS_API
    
    ANTHROPIC_ADAPTER --> OAUTH
    OPENAI_ADAPTER --> OAUTH
    OLLAMA_ADAPTER --> LOCAL
    TOGETHER_ADAPTER --> APIKEY
    FIREWORKS_ADAPTER --> APIKEY
    
    style AGENT fill:#4ecdc4
    style ROUTER_AI fill:#ff6b6b
    style ANTHROPIC_API fill:#f38181
```

## Provider Selection & Failover

```mermaid
sequenceDiagram
    participant Agent
    participant Router as Provider Router
    participant Primary as Primary Provider
    participant Secondary as Secondary Provider
    participant Fallback as Fallback Provider
    participant Cache
    
    Agent->>Router: Send prompt
    Router->>Cache: Check cache
    
    alt Cache hit
        Cache-->>Router: Return cached response
        Router-->>Agent: Return response
    else Cache miss
        Router->>Primary: Try primary provider
        
        alt Primary success
            Primary-->>Router: Return response
            Router->>Cache: Store response
            Router-->>Agent: Return response
        else Primary failure
            Primary-->>Router: Error (rate limit/timeout/auth)
            Router->>Secondary: Try secondary provider
            
            alt Secondary success
                Secondary-->>Router: Return response
                Router->>Cache: Store response
                Router-->>Agent: Return response
            else Secondary failure
                Secondary-->>Router: Error
                Router->>Fallback: Try fallback provider
                
                alt Fallback success
                    Fallback-->>Router: Return response
                    Router->>Cache: Store response
                    Router-->>Agent: Return response
                else All failed
                    Fallback-->>Router: Error
                    Router-->>Agent: Return error
                end
            end
        end
    end
```

## Model Configuration

```mermaid
graph LR
    subgraph "Model Profiles"
        FAST[Fast Model<br/>Sonnet 4.5]
        BALANCED[Balanced Model<br/>Sonnet 4.7]
        SMART[Smart Model<br/>Opus 4.6]
        LOCAL[Local Model<br/>Llama 3.3]
    end
    
    subgraph "Use Cases"
        QUICK[Quick Queries]
        GENERAL[General Chat]
        COMPLEX[Complex Reasoning]
        OFFLINE[Offline/Private]
    end
    
    subgraph "Configuration"
        COST[Cost Optimization]
        LATENCY[Latency Optimization]
        QUALITY[Quality Optimization]
        PRIVACY[Privacy Requirements]
    end
    
    QUICK --> FAST
    GENERAL --> BALANCED
    COMPLEX --> SMART
    OFFLINE --> LOCAL
    
    FAST --> COST
    FAST --> LATENCY
    
    BALANCED --> COST
    BALANCED --> QUALITY
    
    SMART --> QUALITY
    
    LOCAL --> PRIVACY
    LOCAL --> COST
    
    style FAST fill:#51cf66
    style SMART fill:#f38181
    style LOCAL fill:#4ecdc4
```

## OAuth Authentication Flow

```mermaid
sequenceDiagram
    participant User
    participant CLI
    participant Gateway
    participant Browser
    participant Provider as AI Provider<br/>(Anthropic/OpenAI)
    participant Storage
    
    User->>CLI: openclaw login anthropic
    CLI->>Gateway: Request auth URL
    Gateway->>Provider: Initiate OAuth flow
    Provider-->>Gateway: Return auth URL
    Gateway-->>CLI: Return auth URL
    
    CLI->>Browser: Open auth URL
    Browser->>Provider: User login
    User->>Provider: Authenticate
    User->>Provider: Grant permissions
    
    Provider->>Gateway: OAuth callback with code
    Gateway->>Provider: Exchange code for token
    Provider-->>Gateway: Return access token + refresh token
    
    Gateway->>Storage: Store encrypted tokens
    Gateway-->>CLI: Auth successful
    CLI-->>User: Ready to use
```

## Streaming Response Handling

```mermaid
graph TB
    subgraph "Request"
        PROMPT[User Prompt]
        CONTEXT[Conversation Context]
        TOOLS[Available Tools]
    end
    
    subgraph "Provider Stream"
        STREAM[SSE/WebSocket Stream]
        CHUNK[Text Chunks]
        TOOL_CALL[Tool Calls]
        COMPLETE[Completion Signal]
    end
    
    subgraph "Processing"
        BUFFER[Buffer Chunks]
        PARSE[Parse Tool Calls]
        EXECUTE[Execute Tools]
        FORMAT[Format Response]
    end
    
    subgraph "Delivery"
        CHANNEL[Channel Handler]
        DISPLAY[Display to User]
        TYPING[Typing Indicator]
        FINAL[Final Message]
    end
    
    PROMPT --> STREAM
    CONTEXT --> STREAM
    TOOLS --> STREAM
    
    STREAM --> CHUNK
    STREAM --> TOOL_CALL
    STREAM --> COMPLETE
    
    CHUNK --> BUFFER
    TOOL_CALL --> PARSE
    PARSE --> EXECUTE
    EXECUTE --> FORMAT
    BUFFER --> FORMAT
    
    FORMAT --> CHANNEL
    CHANNEL --> TYPING
    CHANNEL --> DISPLAY
    COMPLETE --> FINAL
    
    style STREAM fill:#4ecdc4
    style EXECUTE fill:#ff6b6b
    style CHANNEL fill:#95e1d3
```

## Tool Execution Flow

```mermaid
sequenceDiagram
    participant AI as AI Provider
    participant Agent
    participant Router as Tool Router
    participant Tool
    participant System
    
    AI->>Agent: Request tool call
    Agent->>Agent: Parse tool request
    
    Agent->>Router: Route to tool handler
    Router->>Tool: Execute tool with params
    
    alt System Tool
        Tool->>System: Execute system command
        System-->>Tool: Return result
    else External API
        Tool->>Tool: Call external API
        Tool-->>Tool: Parse response
    else Plugin Tool
        Tool->>Tool: Call plugin function
        Tool-->>Tool: Return result
    end
    
    Tool-->>Router: Return tool result
    Router-->>Agent: Return formatted result
    
    Agent->>AI: Send tool result
    AI->>Agent: Continue generation
    
    alt Needs more tools
        AI->>Agent: Request another tool
        Note over Agent,System: Loop until complete
    else Generation complete
        AI->>Agent: Final response
        Agent->>Agent: Store conversation
    end
```

## Context Window Management

```mermaid
graph TB
    subgraph "Context Sources"
        HISTORY[Message History]
        SYSTEM[System Prompt]
        TOOLS_CTX[Tool Definitions]
        MEMORY_CTX[Retrieved Memory]
    end
    
    subgraph "Context Assembly"
        MEASURE[Measure Tokens]
        PRIORITIZE[Prioritize Content]
        TRUNCATE[Truncate if Needed]
        COMPRESS[Compress History]
    end
    
    subgraph "Window Limits"
        SONNET[Sonnet: 200K tokens]
        OPUS[Opus: 200K tokens]
        GPT4[GPT-4: 128K tokens]
        LOCAL_LIMIT[Local: Model-dependent]
    end
    
    subgraph "Optimization"
        SUMMARIZE[Summarize Old Messages]
        EXTRACT[Extract Key Info]
        STORE[Store to Memory]
        RETRIEVE[Retrieve Relevant]
    end
    
    HISTORY --> MEASURE
    SYSTEM --> MEASURE
    TOOLS_CTX --> MEASURE
    MEMORY_CTX --> MEASURE
    
    MEASURE --> PRIORITIZE
    PRIORITIZE --> TRUNCATE
    TRUNCATE --> COMPRESS
    
    COMPRESS --> SONNET
    COMPRESS --> OPUS
    COMPRESS --> GPT4
    COMPRESS --> LOCAL_LIMIT
    
    HISTORY --> SUMMARIZE
    SUMMARIZE --> EXTRACT
    EXTRACT --> STORE
    STORE --> RETRIEVE
    RETRIEVE --> MEMORY_CTX
    
    style MEASURE fill:#4ecdc4
    style COMPRESS fill:#ff6b6b
    style STORE fill:#95e1d3
```

## Rate Limiting & Throttling

```mermaid
graph LR
    subgraph "Request Queue"
        INCOMING[Incoming Requests]
        QUEUE[Priority Queue]
        SCHEDULER[Request Scheduler]
    end
    
    subgraph "Rate Limits"
        USER_LIMIT[Per-User Limits]
        PROVIDER_LIMIT[Provider Limits]
        CHANNEL_LIMIT[Per-Channel Limits]
    end
    
    subgraph "Throttling"
        TOKEN_BUCKET[Token Bucket]
        BACKOFF[Exponential Backoff]
        RETRY[Retry Logic]
    end
    
    subgraph "Actions"
        ALLOW[Process Request]
        QUEUE_REQ[Queue Request]
        REJECT[Reject Request]
    end
    
    INCOMING --> QUEUE
    QUEUE --> SCHEDULER
    
    SCHEDULER --> USER_LIMIT
    SCHEDULER --> PROVIDER_LIMIT
    SCHEDULER --> CHANNEL_LIMIT
    
    USER_LIMIT --> TOKEN_BUCKET
    PROVIDER_LIMIT --> TOKEN_BUCKET
    CHANNEL_LIMIT --> TOKEN_BUCKET
    
    TOKEN_BUCKET --> ALLOW
    TOKEN_BUCKET --> QUEUE_REQ
    TOKEN_BUCKET --> REJECT
    
    REJECT --> BACKOFF
    BACKOFF --> RETRY
    RETRY --> QUEUE
    
    style SCHEDULER fill:#4ecdc4
    style TOKEN_BUCKET fill:#ff6b6b
    style ALLOW fill:#51cf66
```
