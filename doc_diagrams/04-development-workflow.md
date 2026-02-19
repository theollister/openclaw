# OpenClaw Development Workflow

## Build & Test Pipeline

```mermaid
graph TB
    subgraph "Development"
        CODE[Write Code]
        LINT[Lint & Format]
        TEST[Run Tests]
        BUILD[Build]
    end
    
    subgraph "Pre-commit"
        HOOKS[Git Hooks]
        CHECK[prek install hooks]
        VALIDATE[Validate Changes]
    end
    
    subgraph "CI Pipeline"
        CI[GitHub Actions]
        TSCHECK[Type Check]
        UNITTEST[Unit Tests]
        E2E[E2E Tests]
        COVERAGE[Coverage Check]
    end
    
    subgraph "Release"
        VERSION[Bump Version]
        CHANGELOG[Update Changelog]
        TAG[Create Tag]
        PUBLISH[Publish npm]
        DEPLOY[Deploy Docs]
    end
    
    CODE --> LINT
    LINT --> TEST
    TEST --> BUILD
    
    BUILD --> HOOKS
    HOOKS --> CHECK
    CHECK --> VALIDATE
    
    VALIDATE --> CI
    CI --> TSCHECK
    CI --> UNITTEST
    CI --> E2E
    CI --> COVERAGE
    
    COVERAGE -->|Pass| VERSION
    VERSION --> CHANGELOG
    CHANGELOG --> TAG
    TAG --> PUBLISH
    PUBLISH --> DEPLOY
    
    style CODE fill:#4ecdc4
    style CI fill:#ff6b6b
    style PUBLISH fill:#95e1d3
```

## Build Commands

```mermaid
mindmap
    root((Build System))
        Dependencies
            pnpm install
            bun install
            npm install
        Development
            pnpm dev
            pnpm openclaw
            pnpm tsgo
        Build
            pnpm build
            pnpm build:plugin-sdk:dts
            pnpm canvas:a2ui:bundle
        Quality
            pnpm check
            pnpm lint
            pnpm format
            pnpm check:loc
        Testing
            pnpm test
            pnpm test:coverage
            pnpm test:live
            pnpm test:docker:live-models
        Mobile
            pnpm android:run
            pnpm ios:run
        Documentation
            pnpm docs:dev
            pnpm docs:build
            pnpm docs:check-links
```

## Testing Strategy

```mermaid
graph LR
    subgraph "Unit Tests"
        UNIT[*.test.ts]
        COLOCATED[Colocated with source]
        VITEST[Vitest Runner]
        COV[V8 Coverage]
    end
    
    subgraph "Integration Tests"
        E2E[*.e2e.test.ts]
        DOCKER[Docker Compose]
        GATEWAY[Gateway Tests]
        MODEL[Model Tests]
    end
    
    subgraph "Live Tests"
        LIVE[LIVE=1 tests]
        PROVIDERS[Provider APIs]
        CHANNELS[Channel APIs]
        ONBOARD[Onboarding E2E]
    end
    
    subgraph "Quality Gates"
        LINE[70% Line Coverage]
        BRANCH[70% Branch Coverage]
        FUNC[70% Function Coverage]
        STMT[70% Statement Coverage]
    end
    
    UNIT --> VITEST
    VITEST --> COV
    COV --> LINE
    COV --> BRANCH
    COV --> FUNC
    COV --> STMT
    
    E2E --> DOCKER
    DOCKER --> GATEWAY
    DOCKER --> MODEL
    
    LIVE --> PROVIDERS
    LIVE --> CHANNELS
    LIVE --> ONBOARD
    
    style VITEST fill:#4ecdc4
    style DOCKER fill:#ff6b6b
    style LIVE fill:#ffd93d
```

## Release Channels

```mermaid
graph TB
    subgraph "stable"
        STABLE_TAG[vYYYY.M.D]
        STABLE_NPM[npm: latest]
        STABLE_MAC[macOS App]
    end
    
    subgraph "beta"
        BETA_TAG[vYYYY.M.D-beta.N]
        BETA_NPM[npm: beta]
        BETA_MAC[macOS App Optional]
    end
    
    subgraph "dev"
        DEV_BRANCH[main branch]
        DEV_NPM[npm: dev when published]
        DEV_BUILD[git checkout main]
    end
    
    DEV_BRANCH -->|Release| BETA_TAG
    BETA_TAG -->|Stable| STABLE_TAG
    
    STABLE_TAG --> STABLE_NPM
    STABLE_TAG --> STABLE_MAC
    
    BETA_TAG --> BETA_NPM
    BETA_TAG --> BETA_MAC
    
    DEV_BRANCH --> DEV_NPM
    DEV_BRANCH --> DEV_BUILD
    
    style STABLE_TAG fill:#51cf66
    style BETA_TAG fill:#ffd93d
    style DEV_BRANCH fill:#ff6b6b
```

## Commit Workflow

### Using committer script
```bash
scripts/committer "feat: add new channel" <files>
```

### Conventional Commit Format
- `feat:` - New features
- `fix:` - Bug fixes
- `docs:` - Documentation changes
- `refactor:` - Code refactoring
- `test:` - Test additions/fixes
- `chore:` - Maintenance tasks

### PR Workflow

```mermaid
sequenceDiagram
    participant Dev as Developer
    participant Branch as Feature Branch
    participant PR as Pull Request
    participant Review as Code Review
    participant Main as Main Branch
    
    Dev->>Branch: Create feature branch
    Dev->>Branch: Make changes & commit
    Branch->>PR: Open pull request
    
    PR->>Review: Request review
    Review->>PR: Provide feedback
    
    alt Changes requested
        PR->>Branch: Apply fixes
        Branch->>PR: Push updates
        PR->>Review: Re-review
    end
    
    Review->>PR: Approve
    
    alt Clean commits
        PR->>Main: Rebase & merge
    else Messy commits
        PR->>Main: Squash & merge
    end
    
    Main->>Main: Add changelog entry
    Main->>Main: Run full gate
    Main->>Main: Thank contributor
```

## Platform-Specific Builds

### macOS App
```bash
scripts/package-mac-app.sh  # Current arch
scripts/codesign-mac-app.sh # Sign & notarize
scripts/create-dmg.sh       # Create installer
```

### Mobile Apps
```bash
# iOS
cd apps/ios && xcodebuild ...

# Android
pnpm android:assemble
pnpm android:install
pnpm android:run
```

### Documentation Site
```bash
pnpm docs:dev   # Local preview
pnpm docs:build # Build & check links
```
