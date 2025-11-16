# Module Structure

Walt uses a **multi-module architecture** to achieve separation of concerns, parallel builds, and clear ownership boundaries.

## Directory Structure

```
android/
├── app/                          # Application entry point
│   ├── DI configuration (Hilt)
│   ├── Application class
│   └── AndroidManifest.xml (HCE service declaration)
│
├── feature/                      # Feature modules (presentation layer)
│   ├── provisioning/             # MVI - Card digitization state machine
│   ├── payment/                  # MVI - Tap-to-pay state machine
│   ├── cards/                    # MVVM - Card list and details
│   ├── settings/                 # MVVM - App settings
│   └── onboarding/               # MVVM - User registration
│
├── core/                         # Core modules
│   ├── domain/                   # Business logic layer
│   │   ├── entities              # Domain models (Card, Transaction)
│   │   ├── repositories          # Repository interfaces
│   │   └── usecases              # Business logic operations
│   │
│   ├── common/                   # Shared utilities
│   │   ├── Result sealed interface
│   │   ├── Coroutines utilities
│   │   └── DI qualifiers
│   │
│   ├── data-mtp/                 # MTP SDK wrapper (data layer)
│   │   ├── MTP SDK initialization
│   │   ├── Card digitization
│   │   ├── Token lifecycle management
│   │   └── Repository implementations
│   │
│   ├── data-nfc/                 # NFC/HCE layer (data layer)
│   │   ├── APDU command handling
│   │   ├── Transaction processing
│   │   └── HCE service API
│   │
│   ├── security/                 # Security module
│   │   ├── Android Keystore management
│   │   ├── Play Integrity API
│   │   ├── Root detection
│   │   └── Tamper detection
│   │
│   ├── telemetry/                # Observability (local-only)
│   │   ├── Event tracking
│   │   ├── Debug logging
│   │   └── No PII collection
│   │
│   └── ui/                       # Shared UI components
│       ├── Compose components
│       ├── Theme (Material Design 3)
│       └── Design system
│
├── testing/                      # Test infrastructure
│   ├── fakes/                    # Fake implementations for testing
│   └── contract/                 # Repository contract tests
│
└── build-logic/                  # Build configuration
    └── convention/               # Gradle convention plugins
        ├── AndroidLibraryConventionPlugin
        ├── KotlinLibraryConventionPlugin
        ├── HiltConventionPlugin
        └── QualityConventionPlugin
```

## Module Dependencies

### Allowed Dependencies

```
app → all feature modules, all core modules

feature/provisioning → core/domain, core/data-mtp, core/ui, core/common
feature/payment → core/domain, core/data-nfc, core/ui, core/common
feature/cards → core/domain, core/ui, core/common
feature/settings → core/domain, core/ui, core/common
feature/onboarding → core/domain, core/ui, core/common

core/data-mtp → core/domain, core/security, core/common
core/data-nfc → core/domain, core/security, core/common
core/security → core/common
core/telemetry → core/common
core/ui → core/common
core/domain → NO EXTERNAL DEPENDENCIES (pure Kotlin)

testing/* → any module (for test doubles)
```

### Forbidden Dependencies

- ❌ `core/domain` → Android framework (keeps domain pure)
- ❌ `core/domain` → data layer (dependency inversion)
- ❌ `feature/*` → other `feature/*` (features are isolated)
- ❌ `core/data-mtp` ↔ `core/data-nfc` (no cross-dependencies)

## Why Multi-Module Architecture?

### Separation of Concerns

- `data-mtp` handles payment provider integration
- `data-nfc` handles NFC protocol (independent of provider)
- `security` centralizes security logic for audit compliance
- Features are isolated (can develop/test independently)

### Build Performance

- Gradle can build modules in parallel
- Changes to one module don't rebuild everything
- Faster incremental compilation
- Reduced build times during development

### Code Ownership

- Clear boundaries for who owns what
- Easier to enforce architectural rules
- Simpler code reviews (changes are localized)
- Better collaboration on larger teams

### Testing

- Can test modules in isolation
- Fake implementations per module
- Contract tests ensure interfaces match
- Reduced test scope improves speed

## Module Responsibility Summary

| Module | Responsibility | Dependencies |
|--------|----------------|--------------|
| `app` | Application setup, DI, navigation | All modules |
| `feature/provisioning` | Card digitization UI (MVI) | domain, data-mtp, ui, common |
| `feature/payment` | Tap-to-pay UI (MVI) | domain, data-nfc, ui, common |
| `feature/cards` | Card list/details UI (MVVM) | domain, ui, common |
| `feature/settings` | Settings UI (MVVM) | domain, ui, common |
| `feature/onboarding` | Registration UI (MVVM) | domain, ui, common |
| `core/domain` | Business logic (pure Kotlin) | None |
| `core/common` | Result types, utilities | None |
| `core/data-mtp` | MTP SDK wrapper | domain, security, common |
| `core/data-nfc` | NFC/HCE implementation | domain, security, common |
| `core/security` | Keystore, integrity, root detection | common |
| `core/telemetry` | Local-only logging | common |
| `core/ui` | Compose components, theme | common |
| `testing/fakes` | Test doubles | Any (for testing) |
| `testing/contract` | Repository contract tests | domain |

## Current Implementation Status

### ✅ Complete

- `app/` - Application entry point with Hilt
- `core/common/` - Result sealed interface with 57 tests
- `core/domain/` - Entities and repository interfaces
- `build-logic/convention/` - 4 convention plugins

### 🔄 In Progress

- Repository interfaces being defined

### ⏸️ Planned (awaiting MTP SDK)

- `feature/*` modules
- `core/data-mtp/`
- `core/data-nfc/`
- `core/security/`
- `core/telemetry/`
- `core/ui/`
- `testing/*` modules

## Convention Plugins

Walt uses **Gradle convention plugins** to share configuration across modules:

### Available Plugins

1. **`is.walt.android.library`** - Android library configuration
   - Sets compileSdk 36, minSdk 26
   - Configures Kotlin options
   - Disables unused build features

2. **`is.walt.kotlin.library`** - Pure Kotlin library configuration
   - No Android dependencies
   - Used for `core/domain` module

3. **`is.walt.android.hilt`** - Hilt dependency injection
   - Applies KSP
   - Adds Hilt dependencies
   - Configures annotation processing

4. **`is.walt.quality`** - Code quality tools
   - Applies detekt (static analysis)
   - Applies ktlint (code formatting)
   - Shares configuration across modules

### Usage Example

```kotlin
// core/domain/build.gradle.kts
plugins {
    alias(libs.plugins.walt.kotlin.library)  // Pure Kotlin, no Android
}

// feature/cards/build.gradle.kts (planned)
plugins {
    alias(libs.plugins.walt.android.library)  // Android library
    alias(libs.plugins.walt.android.hilt)     // Hilt DI
    alias(libs.plugins.walt.quality)          // Quality checks
}
```

**See**: [Build System Documentation](build-system.md) for details.

## Naming Conventions

### Package Names

- **Application**: `is.walt.app`
- **Core modules**: `is.walt.core.<module>` (e.g., `is.walt.core.domain`)
- **Feature modules**: `is.walt.feature.<feature>` (e.g., `is.walt.feature.cards`)
- **Testing modules**: `is.walt.testing.<module>` (e.g., `is.walt.testing.fakes`)

### Module Names

- **Core**: `core/<name>` (e.g., `core/domain`, `core/common`)
- **Features**: `feature/<name>` (e.g., `feature/cards`, `feature/provisioning`)
- **Testing**: `testing/<name>` (e.g., `testing/fakes`)

## Gradle Configuration

All modules are registered in `settings.gradle.kts`:

```kotlin
rootProject.name = "Walt"
include(":app")
include(":core:common")
include(":core:domain")
// Future modules will be added here
```

Dependencies are managed via version catalog (`gradle/libs.versions.toml`).

**See**: [Build System Documentation](build-system.md)

## Related Documentation

- [Domain Layer](domain-layer.md) - Business logic architecture
- [Data Layer](data-layer.md) - External integrations
- [Presentation Layer](presentation-layer.md) - UI architecture
- [Build System](build-system.md) - Gradle configuration

---

**Back to**: [Android Implementation Overview](README.md)
