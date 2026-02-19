# OpenClaw Mobile Architecture

## iOS App Architecture

```mermaid
graph TB
    subgraph "UI Layer (SwiftUI)"
        MAIN[Main View]
        CHAT[Chat View]
        SETTINGS[Settings View]
        CANVAS[Canvas View]
        VOICE[Voice Interface]
    end
    
    subgraph "View Models (Observation)"
        CHAT_VM[Chat ViewModel]
        SETTINGS_VM[Settings ViewModel]
        CANVAS_VM[Canvas ViewModel]
        VOICE_VM[Voice ViewModel]
    end
    
    subgraph "Services"
        GATEWAY_CLIENT[Gateway Client]
        WEBSOCKET[WebSocket Manager]
        AUDIO[Audio Manager]
        MEDIA_IOS[Media Manager]
        KEYCHAIN[Keychain Service]
    end
    
    subgraph "Gateway Connection"
        HTTP[HTTP API]
        WS[WebSocket]
        PAIRING_IOS[Device Pairing]
    end
    
    MAIN --> CHAT
    MAIN --> SETTINGS
    MAIN --> CANVAS
    MAIN --> VOICE
    
    CHAT --> CHAT_VM
    SETTINGS --> SETTINGS_VM
    CANVAS --> CANVAS_VM
    VOICE --> VOICE_VM
    
    CHAT_VM --> GATEWAY_CLIENT
    SETTINGS_VM --> GATEWAY_CLIENT
    CANVAS_VM --> GATEWAY_CLIENT
    VOICE_VM --> AUDIO
    
    GATEWAY_CLIENT --> HTTP
    GATEWAY_CLIENT --> WS
    GATEWAY_CLIENT --> PAIRING_IOS
    
    AUDIO --> WEBSOCKET
    MEDIA_IOS --> WEBSOCKET
    SETTINGS_VM --> KEYCHAIN
    
    style MAIN fill:#4ecdc4
    style GATEWAY_CLIENT fill:#ff6b6b
    style CHAT_VM fill:#95e1d3
```

## Android App Architecture

```mermaid
graph TB
    subgraph "Presentation Layer (Jetpack Compose)"
        MAIN_SCREEN[MainActivity]
        CHAT_SCREEN[Chat Screen]
        SETTINGS_SCREEN[Settings Screen]
        CANVAS_SCREEN[Canvas Screen]
        VOICE_SCREEN[Voice Screen]
    end
    
    subgraph "ViewModels (AAC)"
        CHAT_VM_ANDROID[ChatViewModel]
        SETTINGS_VM_ANDROID[SettingsViewModel]
        CANVAS_VM_ANDROID[CanvasViewModel]
        VOICE_VM_ANDROID[VoiceViewModel]
    end
    
    subgraph "Domain Layer"
        USECASES[Use Cases]
        REPO[Repository]
        MODELS[Data Models]
    end
    
    subgraph "Data Layer"
        GATEWAY_API[Gateway API Client]
        WEBSOCKET_ANDROID[WebSocket Client]
        LOCAL_DB[Local Database (Room)]
        PREFS[SharedPreferences]
    end
    
    subgraph "Services"
        AUDIO_SERVICE[Audio Service]
        MEDIA_SERVICE[Media Service]
        NOTIFICATION[Notification Manager]
    end
    
    MAIN_SCREEN --> CHAT_SCREEN
    MAIN_SCREEN --> SETTINGS_SCREEN
    MAIN_SCREEN --> CANVAS_SCREEN
    MAIN_SCREEN --> VOICE_SCREEN
    
    CHAT_SCREEN --> CHAT_VM_ANDROID
    SETTINGS_SCREEN --> SETTINGS_VM_ANDROID
    CANVAS_SCREEN --> CANVAS_VM_ANDROID
    VOICE_SCREEN --> VOICE_VM_ANDROID
    
    CHAT_VM_ANDROID --> USECASES
    SETTINGS_VM_ANDROID --> USECASES
    CANVAS_VM_ANDROID --> USECASES
    VOICE_VM_ANDROID --> USECASES
    
    USECASES --> REPO
    REPO --> MODELS
    
    REPO --> GATEWAY_API
    REPO --> WEBSOCKET_ANDROID
    REPO --> LOCAL_DB
    REPO --> PREFS
    
    GATEWAY_API --> AUDIO_SERVICE
    GATEWAY_API --> MEDIA_SERVICE
    GATEWAY_API --> NOTIFICATION
    
    style MAIN_SCREEN fill:#4ecdc4
    style REPO fill:#ff6b6b
    style GATEWAY_API fill:#95e1d3
```

## macOS App Architecture

```mermaid
graph TB
    subgraph "UI Layer (SwiftUI)"
        MENUBAR[Menubar App]
        STATUS[Status Window]
        CHAT_MAC[Chat Window]
        SETTINGS_MAC[Settings Window]
        ONBOARDING[Onboarding Flow]
    end
    
    subgraph "App Services"
        GATEWAY_MANAGER[Gateway Manager]
        LAUNCHD_MANAGER[LaunchAgent Manager]
        CONFIG_MANAGER[Config Manager]
        VOICE_WAKE[Voice Wake Forwarder]
    end
    
    subgraph "Gateway Integration"
        EMBEDDED_GW[Embedded Gateway]
        CLI_BRIDGE[CLI Bridge]
        IPC[IPC Communication]
    end
    
    subgraph "System Integration"
        KEYCHAIN_MAC[Keychain]
        NOTIF_CENTER[Notification Center]
        OSLOG[Unified Logging]
        SPEECH[Speech Recognition]
    end
    
    MENUBAR --> STATUS
    MENUBAR --> CHAT_MAC
    MENUBAR --> SETTINGS_MAC
    MENUBAR --> ONBOARDING
    
    STATUS --> GATEWAY_MANAGER
    CHAT_MAC --> GATEWAY_MANAGER
    SETTINGS_MAC --> CONFIG_MANAGER
    ONBOARDING --> CONFIG_MANAGER
    
    GATEWAY_MANAGER --> EMBEDDED_GW
    GATEWAY_MANAGER --> CLI_BRIDGE
    GATEWAY_MANAGER --> IPC
    GATEWAY_MANAGER --> LAUNCHD_MANAGER
    
    EMBEDDED_GW --> VOICE_WAKE
    VOICE_WAKE --> SPEECH
    
    CONFIG_MANAGER --> KEYCHAIN_MAC
    GATEWAY_MANAGER --> NOTIF_CENTER
    GATEWAY_MANAGER --> OSLOG
    
    style MENUBAR fill:#4ecdc4
    style EMBEDDED_GW fill:#ff6b6b
    style GATEWAY_MANAGER fill:#95e1d3
```

## Shared Code Architecture

```mermaid
graph LR
    subgraph "Platform Apps"
        IOS_APP[iOS App]
        ANDROID_APP[Android App]
        MACOS_APP[macOS App]
    end
    
    subgraph "Shared Code (apps/shared)"
        MODELS_SHARED[Data Models]
        NETWORK[Network Layer]
        UTILS_SHARED[Utilities]
        UI_COMMON[Common UI Components]
    end
    
    subgraph "Platform-Specific"
        IOS_SPECIFIC[iOS-specific Code]
        ANDROID_SPECIFIC[Android-specific Code]
        MACOS_SPECIFIC[macOS-specific Code]
    end
    
    IOS_APP --> MODELS_SHARED
    IOS_APP --> NETWORK
    IOS_APP --> UTILS_SHARED
    IOS_APP --> UI_COMMON
    IOS_APP --> IOS_SPECIFIC
    
    ANDROID_APP --> MODELS_SHARED
    ANDROID_APP --> NETWORK
    ANDROID_APP --> UTILS_SHARED
    ANDROID_APP --> ANDROID_SPECIFIC
    
    MACOS_APP --> MODELS_SHARED
    MACOS_APP --> NETWORK
    MACOS_APP --> UTILS_SHARED
    MACOS_APP --> UI_COMMON
    MACOS_APP --> MACOS_SPECIFIC
    
    style MODELS_SHARED fill:#95e1d3
    style NETWORK fill:#4ecdc4
    style IOS_APP fill:#ff6b6b
```

## Mobile Gateway Communication

```mermaid
sequenceDiagram
    participant App as Mobile App
    participant WS as WebSocket
    participant Gateway
    participant Channel
    participant AI
    
    App->>Gateway: Connect (HTTP handshake)
    Gateway-->>App: Return session token
    
    App->>WS: Establish WebSocket
    WS-->>App: Connection confirmed
    
    App->>WS: Send message
    WS->>Gateway: Forward message
    Gateway->>Channel: Route to channel
    Channel->>AI: Process with AI
    AI-->>Channel: Stream response
    
    loop Streaming Response
        Channel->>Gateway: Stream chunk
        Gateway->>WS: Forward chunk
        WS-->>App: Display chunk
    end
    
    AI-->>Channel: Complete
    Channel->>Gateway: Final message
    Gateway->>WS: Completion signal
    WS-->>App: Update UI
```

## Voice Interface Flow

```mermaid
graph TB
    subgraph "Voice Input"
        MIC[Microphone]
        RECORD[Audio Recording]
        ENCODE[Audio Encoding]
    end
    
    subgraph "Speech Recognition"
        STT[Speech-to-Text]
        TRANSCRIPT[Transcript]
        CONFIDENCE[Confidence Score]
    end
    
    subgraph "Processing"
        GATEWAY_VOICE[Gateway Processing]
        AI_VOICE[AI Response]
        TTS[Text-to-Speech]
    end
    
    subgraph "Voice Output"
        SYNTHESIS[Voice Synthesis]
        DECODE[Audio Decoding]
        SPEAKER[Speaker Output]
    end
    
    MIC --> RECORD
    RECORD --> ENCODE
    ENCODE --> STT
    
    STT --> TRANSCRIPT
    STT --> CONFIDENCE
    
    TRANSCRIPT --> GATEWAY_VOICE
    GATEWAY_VOICE --> AI_VOICE
    AI_VOICE --> TTS
    
    TTS --> SYNTHESIS
    SYNTHESIS --> DECODE
    DECODE --> SPEAKER
    
    style STT fill:#4ecdc4
    style AI_VOICE fill:#ff6b6b
    style TTS fill:#95e1d3
```

## Canvas Integration

```mermaid
graph LR
    subgraph "Mobile Canvas"
        CANVAS_UI[Canvas View]
        WEBVIEW[WebView/WKWebView]
        A2UI[A2UI Runtime]
    end
    
    subgraph "Gateway Canvas Host"
        CANVAS_SERVER[Canvas Server]
        A2UI_BUNDLE[A2UI Bundle]
        STATE[Canvas State]
    end
    
    subgraph "Interaction"
        GESTURES[Touch Gestures]
        UPDATES[State Updates]
        RENDER[Rendering]
    end
    
    CANVAS_UI --> WEBVIEW
    WEBVIEW --> A2UI
    
    A2UI --> CANVAS_SERVER
    CANVAS_SERVER --> A2UI_BUNDLE
    CANVAS_SERVER --> STATE
    
    GESTURES --> UPDATES
    UPDATES --> STATE
    STATE --> RENDER
    RENDER --> CANVAS_UI
    
    style WEBVIEW fill:#4ecdc4
    style CANVAS_SERVER fill:#ff6b6b
    style A2UI fill:#95e1d3
```

## Push Notifications

```mermaid
sequenceDiagram
    participant Gateway
    participant APNS as APNs/FCM
    participant Device
    participant App
    
    Gateway->>Gateway: New message arrives
    Gateway->>Gateway: Check device tokens
    
    alt iOS Device
        Gateway->>APNS: Send to APNs
        APNS->>Device: Push notification
    else Android Device
        Gateway->>APNS: Send to FCM
        APNS->>Device: Push notification
    end
    
    Device->>App: Wake app
    App->>App: Display notification
    
    alt User taps notification
        App->>App: Open chat
        App->>Gateway: Request full message
        Gateway-->>App: Return message
        App->>App: Display in UI
    end
```

## Offline Support

```mermaid
graph TB
    subgraph "Online Mode"
        ONLINE[Connected to Gateway]
        REALTIME[Real-time Messages]
        SYNC[Auto-sync]
    end
    
    subgraph "Offline Mode"
        OFFLINE[No Connection]
        QUEUE[Message Queue]
        LOCAL_STORAGE[Local Cache]
    end
    
    subgraph "Sync Logic"
        DETECT[Detect Connection]
        MERGE[Merge Changes]
        RESOLVE[Conflict Resolution]
        PERSIST[Persist to Server]
    end
    
    subgraph "User Experience"
        INDICATOR[Offline Indicator]
        DRAFT[Draft Messages]
        READ[Read Cached Messages]
    end
    
    ONLINE --> REALTIME
    ONLINE --> SYNC
    
    OFFLINE --> QUEUE
    OFFLINE --> LOCAL_STORAGE
    OFFLINE --> INDICATOR
    
    QUEUE --> DRAFT
    LOCAL_STORAGE --> READ
    
    DETECT --> MERGE
    MERGE --> RESOLVE
    RESOLVE --> PERSIST
    PERSIST --> ONLINE
    
    OFFLINE -.Reconnect.-> DETECT
    
    style ONLINE fill:#51cf66
    style OFFLINE fill:#ffd93d
    style DETECT fill:#4ecdc4
```
