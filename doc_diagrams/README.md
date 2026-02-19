# OpenClaw Documentation Diagrams

This folder contains comprehensive architecture and system documentation diagrams for the OpenClaw project.

## � Start Here

### [Complete System Overview](00-complete-overview.md) ⭐
**Master diagram showing all subsystems and their interactions** - Start here for the big picture!

This overview provides:
- Master system diagram with all components
- System interaction patterns
- Key architectural principles
- Component responsibilities
- Quick reference guide

## �📚 Complete Documentation Set

### 1. [System Architecture](01-system-architecture.md)
High-level overview of the entire OpenClaw system showing major components and their relationships:
- Gateway service and core services
- Channel integrations (built-in and extensions)
- AI provider connections
- Mobile and desktop applications
- Infrastructure components

**Key Diagrams**: System overview, component descriptions

### 2. [Repository Structure](02-repository-structure.md)
Detailed breakdown of the monorepo organization:
- `/src` - Core source code organization
- `/apps` - Platform applications (macOS, iOS, Android)
- `/extensions` - Plugin system and extensions
- `/docs` - Documentation structure
- Build scripts and infrastructure

**Key Diagrams**: Directory tree, module organization

### 3. [Message Flow Architecture](03-message-flow.md)
How messages flow through the system:
- Message routing and processing
- Channel registration and discovery
- Routing rules and allowlists
- Multi-channel support (all built-in and extension channels)

**Key Diagrams**: Sequence diagrams, routing flow, channel lifecycle

### 4. [Development Workflow](04-development-workflow.md)
Build, test, and release processes:
- Build pipeline and CI/CD
- Testing strategy (unit, integration, live tests)
- Release channels (stable, beta, dev)
- Commit and PR workflow
- Platform-specific builds

**Key Diagrams**: Build flow, testing pyramid, release pipeline, PR workflow

### 5. [Plugin Architecture](05-plugin-architecture.md)
Extension and plugin system design:
- Plugin SDK and API surface
- Plugin discovery and loading
- Extension package structure
- Plugin types (channel, auth, memory, tool, feature)
- Security and isolation

**Key Diagrams**: Plugin lifecycle, class diagrams, dependency flow

### 6. [Data Flow Architecture](06-data-flow.md)
Configuration, credentials, and data management:
- Configuration system and merging
- Credential storage and OAuth flows
- Session and memory management
- Media pipeline processing
- Logging and observability
- Device pairing and sync
- Cache strategies

**Key Diagrams**: Config flow, encryption, media pipeline, pairing

### 7. [Deployment Architecture](07-deployment-architecture.md)
Different deployment scenarios and platforms:
- Local deployment (macOS, Pi, Linux, Windows WSL2)
- Cloud deployment (Fly.io, Docker, VPS, Kubernetes)
- Hybrid deployment with tunneling
- Platform-specific deployment guides
- Network architecture
- High availability setup

**Key Diagrams**: Deployment options, Pi setup, Docker stack, Fly.io config

### 8. [AI Provider Integration](08-ai-provider-integration.md)
Multi-provider AI system design:
- Provider adapters and routing
- Provider selection and failover
- Model configuration and profiles
- OAuth authentication flows
- Streaming response handling
- Tool execution
- Context window management
- Rate limiting and throttling

**Key Diagrams**: Provider flow, failover sequence, streaming, context management

### 9. [Mobile Architecture](09-mobile-architecture.md)
iOS, Android, and macOS application design:
- iOS app architecture (SwiftUI + Observation)
- Android app architecture (Jetpack Compose + AAC)
- macOS menubar app integration
- Shared code organization
- Gateway communication via WebSocket
- Voice interface flow
- Canvas integration
- Push notifications
- Offline support

**Key Diagrams**: App layer diagrams, communication sequences, voice flow

### 10. [Security Architecture](10-security-architecture.md)
Comprehensive security design:
- Security layers overview
- Device pairing security
- Credential storage (platform-specific)
- Allowlist and authorization
- Encryption at rest and in transit
- Vulnerability protection
- Privacy and data handling
- Audit and compliance

**Key Diagrams**: Security layers, pairing sequence, credential storage, threat mitigation

### 11. [Quick Reference Guide](11-quick-reference.md) 🔧
Practical guide for common development tasks:
- Adding new channels and providers
- Creating AI tools
- Debugging message flow
- Deploying to Raspberry Pi
- Creating plugin extensions
- Testing strategies
- Release workflow
- Troubleshooting checklist

**Key Diagrams**: Task flows, patterns, code examples

## 🎨 Diagram Types Used

### Mermaid Diagrams
All diagrams use Mermaid syntax for easy versioning and rendering:

- **Graph/Flowchart**: System architecture, data flow, component relationships
- **Sequence Diagrams**: Message passing, API calls, authentication flows
- **Class Diagrams**: Plugin API, object relationships
- **Mindmaps**: Build system, concept hierarchies

### Rendering Options

**GitHub**: Native Mermaid support in markdown files (view directly in repo)

**VS Code**: Install [Mermaid extension](https://marketplace.visualstudio.com/items?itemName=bierner.markdown-mermaid)

**Online Editor**: [mermaid.live](https://mermaid.live/) for editing and exporting

**Documentation Sites**: Mintlify, MkDocs, and other platforms support Mermaid natively

## 📖 Usage Guide

### For New Contributors
Start with these diagrams to understand the project:
1. [System Architecture](01-system-architecture.md) - Big picture
2. [Repository Structure](02-repository-structure.md) - Code organization
3. [Development Workflow](04-development-workflow.md) - How to build and test

### For Feature Development
Understand specific subsystems:
- Adding channels? See [Message Flow](03-message-flow.md)
- Building plugins? See [Plugin Architecture](05-plugin-architecture.md)
- Mobile work? See [Mobile Architecture](09-mobile-architecture.md)
- Deployment? See [Deployment Architecture](07-deployment-architecture.md)

### For Architecture Review
Comprehensive system understanding:
- [AI Provider Integration](08-ai-provider-integration.md) - Provider system design
- [Data Flow](06-data-flow.md) - How data moves through the system
- [Security Architecture](10-security-architecture.md) - Security patterns and practices

## 🔄 Maintenance

These diagrams are living documentation and should be updated when:
- Major architectural changes occur
- New subsystems or components are added
- Deployment patterns change
- Security practices evolve

When updating diagrams, ensure:
1. Mermaid syntax remains valid
2. Descriptions stay accurate
3. Cross-references to other diagrams are maintained
4. Color scheme stays consistent

## 🎯 Diagram Conventions

### Color Coding
- **Blue (#4ecdc4)**: Primary components, main flows
- **Red (#ff6b6b)**: Critical components, gateways, core services
- **Green (#51cf66, #95e1d3)**: Success states, plugins, extensions
- **Yellow (#ffd93d)**: Warnings, queues, processing states
- **Purple (#aa96da, #f38181)**: External services, AI providers

### Naming
- Clear, descriptive node labels
- Consistent terminology across diagrams
- Acronyms explained in context

## 📝 Contributing Diagrams

When adding new diagrams:
1. Use Mermaid syntax for consistency
2. Number files sequentially (`##-descriptive-name.md`)
3. Include diagram title and context
4. Add color styling for visual hierarchy
5. Update this README index
6. Cross-reference related diagrams

## 🔗 Related Documentation

- Main docs: `docs/` directory
- Code architecture: See inline code comments
- API reference: `docs/reference/`
- Setup guides: `docs/install/`
