# Technology Stack

This document details all technologies, frameworks, and tools used in Walt's Android implementation.

## Language

### Kotlin 100%

**Version**: 2.2.21

Walt is written entirely in Kotlin with **no Java code**:

- ✅ Modern, concise syntax
- ✅ Null safety built into the language
- ✅ Coroutines for async operations
- ✅ Flow for reactive streams
- ✅ Official language for Android development
- ✅ Jetpack Compose requires Kotlin
- ✅ Better tooling and IDE support

**Rationale**: Java is legacy for Android. Google recommends Kotlin for all new projects.

---

## UI Framework

### Jetpack Compose

**Version**: BOM 2024.09.00

**What it is**: Modern declarative UI toolkit for Android

**Why Compose?**:
- Declarative UI (describe what, not how)
- Less boilerplate than XML layouts
- Type-safe (compile-time checks)
- Hot reload for faster iteration
- Better state management
- Official Android recommendation

**Example**:

```kotlin
@Composable
fun CardsList(cards: List<Card>) {
    LazyColumn {
        items(cards) { card ->
            CardListItem(card = card)
        }
    }
}
```

### Material Design 3

**What it is**: Google's latest design system

**Features**:
- Dynamic color (adapts to user wallpaper)
- Dark mode support
- Accessibility built-in
- Modern, consistent UI components

**Implementation**:

```kotlin
@Composable
fun WaltTheme(
    darkTheme: Boolean = isSystemInDarkTheme(),
    dynamicColor: Boolean = true,
    content: @Composable () -> Unit
) {
    val colorScheme = when {
        dynamicColor && Build.VERSION.SDK_INT >= Build.VERSION_CODES.S -> {
            val context = LocalContext.current
            if (darkTheme) dynamicDarkColorScheme(context)
            else dynamicLightColorScheme(context)
        }
        darkTheme -> DarkColorScheme
        else -> LightColorScheme
    }

    MaterialTheme(
        colorScheme = colorScheme,
        typography = Typography,
        content = content
    )
}
```

---

## Architecture

### Clean Architecture

**Principles**:
- Separation of concerns (domain/data/presentation)
- Dependency inversion (data depends on domain)
- Testability (pure business logic)

**See**: [Domain Layer](domain-layer.md), [Data Layer](data-layer.md), [Presentation Layer](presentation-layer.md)

### MVVM + MVI Hybrid

**MVVM** (Model-View-ViewModel):
- Used for simple features (cards, settings)
- Less boilerplate
- Natural fit with Compose

**MVI** (Model-View-Intent):
- Used for state machines (provisioning, payment)
- Explicit states and events
- Audit trail for critical flows

**See**: [Presentation Layer](presentation-layer.md)

---

## Dependency Injection

### Hilt

**Version**: 2.57.2

**What it is**: Dependency injection framework built on Dagger, optimized for Android

**Why Hilt over Koin?**:
- ✅ **Compile-time safety** - Catches errors at build time
- ✅ **Android-optimized** - Built by Google for Android
- ✅ **Multi-module support** - Works with Walt's module structure
- ✅ **Industry standard** - Based on Dagger
- ✅ **Better IDE support** - Clear error messages

vs **Koin**:
- ❌ Runtime injection (crashes at runtime, not build time)
- ❌ Less type safety

**For a payment app, compile-time safety is critical.**

**Example**:

```kotlin
// Application class
@HiltAndroidApp
class WaltApplication : Application()

// Module
@Module
@InstallIn(SingletonComponent::class)
interface DataModule {
    @Binds
    fun bindCardRepository(
        impl: CardRepositoryImpl
    ): CardRepository
}

// ViewModel
@HiltViewModel
class CardsViewModel @Inject constructor(
    private val getCardsUseCase: GetCardsUseCase
) : ViewModel()

// Compose
@Composable
fun CardsScreen(
    viewModel: CardsViewModel = hiltViewModel()
) {
    // ViewModel automatically injected
}
```

### KSP (Kotlin Symbol Processing)

**Version**: 2.3.2

**What it is**: Annotation processing for Kotlin (replacement for kapt)

**Why KSP over kapt?**:
- ✅ **2x faster** build times
- ✅ Better Kotlin support
- ✅ Official replacement for kapt
- ✅ Hilt now supports KSP

**Used by**: Hilt, Room (planned)

---

## Asynchronous Programming

### Kotlin Coroutines

**Version**: 1.10.2

**What it is**: Kotlin's solution for async/await operations

**Why Coroutines?**:
- ✅ Structured concurrency (scoped lifecycles)
- ✅ Cancellation support
- ✅ Sequential code that runs async
- ✅ Better than callbacks (no callback hell)
- ✅ Integration with Android lifecycle

**Example**:

```kotlin
class CardsViewModel @Inject constructor(
    private val getCardsUseCase: GetCardsUseCase
) : ViewModel() {

    private fun loadCards() {
        viewModelScope.launch {  // Cancelled when ViewModel cleared
            getCardsUseCase().collect { result ->
                // Handle result
            }
        }
    }
}
```

### Kotlin Flow

**Version**: 1.10.2 (part of coroutines library)

**What it is**: Reactive streams for Kotlin

**Why Flow over RxJava?**:
- ✅ Native Kotlin support
- ✅ Coroutines integration
- ✅ Simpler API
- ✅ Cold streams by default
- ✅ Better for Android (lifecycle-aware)

**Example**:

```kotlin
interface CardRepository {
    fun getCards(): Flow<Result<List<Card>>>
}

// Usage
cardRepository.getCards()
    .map { result -> /* transform */ }
    .catch { e -> /* handle error */ }
    .collect { result -> /* use data */ }
```

---

## State Management

### StateFlow

**Version**: 1.10.2 (part of coroutines library)

**What it is**: Hot Flow optimized for state management

**Why StateFlow over LiveData?**:
- ✅ Modern replacement for LiveData
- ✅ Better Kotlin coroutines integration
- ✅ More flexible operators
- ✅ Lifecycle-aware in Compose (`collectAsStateWithLifecycle`)
- ✅ LiveData is in maintenance mode

**Example**:

```kotlin
// ViewModel
class CardsViewModel @Inject constructor() : ViewModel() {
    private val _state = MutableStateFlow(CardsState())
    val state: StateFlow<CardsState> = _state.asStateFlow()

    fun updateState() {
        _state.update { it.copy(isLoading = true) }
    }
}

// Compose
@Composable
fun CardsScreen(viewModel: CardsViewModel = hiltViewModel()) {
    val state by viewModel.state.collectAsStateWithLifecycle()
    // UI automatically recomposes when state changes
}
```

---

## Testing

### JUnit 4

**Version**: 4.13.2

**What it is**: Unit testing framework

**Why JUnit 4 over JUnit 5?**:
- ✅ Better Android tooling support
- ✅ Official Android recommendation
- ✅ Simpler setup (no platform dependencies)
- ✅ All Android documentation uses JUnit 4

**Example**:

```kotlin
class CardRepositoryTest {
    @Test
    fun `getCards returns empty list initially`() {
        val repository = FakeCardRepository()
        val result = runBlocking { repository.getCards().first() }
        assertThat(result).isInstanceOf(Result.Success::class.java)
    }
}
```

### Google Truth

**Version**: 1.4.5

**What it is**: Fluent assertion library

**Why Truth over JUnit assertions?**:
- ✅ More readable assertions
- ✅ Better error messages
- ✅ Fluent API
- ✅ Built by Google for Android

**Example**:

```kotlin
// Google Truth (chosen)
assertThat(cards).hasSize(3)
assertThat(card.state).isEqualTo(CardState.Active)
assertThat(result).isInstanceOf(Result.Success::class.java)

// vs JUnit assertions (verbose)
assertEquals(3, cards.size)
assertEquals(CardState.Active, card.state)
assertTrue(result is Result.Success)
```

### Coroutines Test

**Version**: 1.10.2

**What it is**: Testing utilities for coroutines and Flow

**Example**:

```kotlin
@Test
fun `loadCards updates state`() = runTest {
    val viewModel = CardsViewModel(fakeUseCase)
    viewModel.loadCards()

    val state = viewModel.state.value
    assertThat(state.cards).hasSize(3)
}
```

### Testing Strategy

- **Fakes over mocks** - Reusable, realistic test doubles
- **Contract tests** - Ensure fakes match real implementations
- **TDD approach** - Tests written first when possible

**See**: [Quality Gates](quality-gates.md)

---

## Payment Infrastructure

### MeaWallet MTP SDK

**Status**: ⏸️ Awaiting SDK access

**What it is**: Mobile Token Platform for card tokenization and payments

**What MTP SDK provides**:
- ✅ Card tokenization (DPAN creation)
- ✅ NFC/HCE contactless payments
- ✅ Token lifecycle management
- ✅ Card network integration (Visa/Mastercard)
- ✅ Push notification handling (FCM)

**What Walt implements**:
- UI for provisioning and payment (Jetpack Compose)
- Local transaction storage (Room + encryption)
- Additional security hardening
- Privacy-focused data handling

**See**: [Card Loading Protocol](../card-loading/README.md), [Data Layer](data-layer.md)

---

## Security

### Android Keystore

**What it is**: Hardware-backed key storage on Android

**Features**:
- Keys stored in TEE (Trusted Execution Environment)
- StrongBox support (separate security chip)
- Keys never leave hardware
- Biometric authentication for key use

**Usage**: Encrypt transaction data, secure token storage

### Play Integrity API

**What it is**: Google's device and app verification service

**Usage**:
- Device attestation during card provisioning
- Verify app hasn't been tampered with
- Periodic integrity checks

### SQLCipher (Planned)

**What it is**: Encrypted SQLite database

**Usage**: Encrypt Room database for local storage

---

## Build System

### Gradle

**Version**: 8.13 (wrapper)

**Android Gradle Plugin**: 8.13.1

**Features**:
- Multi-module builds
- Parallel builds
- Incremental compilation
- Convention plugins for shared configuration

**See**: [Build System](build-system.md)

### Version Catalog

**File**: `gradle/libs.versions.toml`

**What it is**: Centralized dependency management

**Benefits**:
- Single source of truth for versions
- Type-safe accessors
- Easy global updates
- IDE support

**Example**:

```toml
[versions]
kotlin = "2.2.21"
hilt = "2.57.2"

[libraries]
hilt-android = { group = "com.google.dagger", name = "hilt-android", version.ref = "hilt" }

[plugins]
ksp = { id = "com.google.devtools.ksp", version.ref = "ksp" }
```

---

## Code Quality

### detekt

**Version**: 1.23.8

**What it is**: Static analysis for Kotlin

**Detects**:
- Security issues (hardcoded secrets, unsafe crypto)
- Potential bugs (null safety violations)
- Code smells (complexity, naming)

**Usage**: `./gradlew detekt`

### ktlint

**Version**: 12.1.1

**What it is**: Code formatting and style enforcement

**Benefits**:
- Consistent code style
- Prevents formatting debates
- Auto-fix available (opt-in)

**Usage**: `./gradlew ktlintCheck`

**See**: [Quality Gates](quality-gates.md)

---

## CI/CD

### GitHub Actions

**What it is**: Automated CI/CD pipeline

**Workflow** (`.github/workflows/quality.yml`):
- Runs on every PR and push to main
- Executes: `./gradlew build detekt ktlintCheck`
- Fails PR if violations found

**See**: [Quality Gates](quality-gates.md)

### Dependabot

**What it is**: Automated dependency updates

**Configuration**:
- Weekly checks for Gradle dependencies
- Auto-labels by update type (major/minor/patch)
- Groups updates for easier review

---

## SDK Requirements

### Minimum SDK: API 26 (Android 8.0)

**Released**: August 2017

**Coverage**: ~98% of Android devices

**Why API 26?**:
- ✅ Modern Android features (notification channels, autofill)
- ✅ Sufficient for NFC/HCE (available since API 19)
- ✅ Avoids legacy baggage
- ✅ Still reaches vast majority of users

### Target SDK: API 36 (Android 15)

**Updated**: November 2025

**Why API 36?**:
- ✅ Google Play Store requirements (targetSdk 34+ mandatory)
- ✅ Latest stable Android version
- ✅ androidx.core 1.17.0+ requires compileSdk 36
- ✅ Ecosystem moving to API 36
- ✅ Early in development (best time for updates)

### Compile SDK: API 36 (Android 15)

**Rationale**: Always match compileSdk with targetSdk for consistency

---

## Development Tools

### Android Studio

**Recommended Version**: Latest stable (Ladybug or newer)

**Features**:
- Kotlin support
- Jetpack Compose preview
- Hilt integration
- Gradle build system

### JDK 17

**Why JDK 17?**:
- Required for Android Gradle Plugin 8.0+
- LTS (Long Term Support) version
- Modern Java features

---

## Summary Table

| Category | Technology | Version | Purpose |
|----------|-----------|---------|---------|
| **Language** | Kotlin | 2.2.21 | Primary language (100%) |
| **UI** | Jetpack Compose | BOM 2024.09.00 | Declarative UI |
| | Material Design 3 | Latest | Design system |
| **Architecture** | Clean Architecture | - | Separation of concerns |
| | MVVM + MVI | - | Presentation patterns |
| **DI** | Hilt | 2.57.2 | Dependency injection |
| | KSP | 2.3.2 | Annotation processing |
| **Async** | Kotlin Coroutines | 1.10.2 | Async operations |
| | Kotlin Flow | 1.10.2 | Reactive streams |
| **State** | StateFlow | 1.10.2 | UI state management |
| **Testing** | JUnit 4 | 4.13.2 | Unit tests |
| | Google Truth | 1.4.5 | Assertions |
| | Coroutines Test | 1.10.2 | Coroutine testing |
| **Payment** | MeaWallet MTP SDK | TBD | Card tokenization |
| **Security** | Android Keystore | OS | Encryption keys |
| | Play Integrity API | Latest | Device attestation |
| **Build** | Gradle | 8.13 | Build system |
| | Android Gradle Plugin | 8.13.1 | Android builds |
| | Version Catalog | - | Dependency management |
| **Quality** | detekt | 1.23.8 | Static analysis |
| | ktlint | 12.1.1 | Code formatting |
| **CI/CD** | GitHub Actions | Latest | Automated testing |
| | Dependabot | Latest | Dependency updates |

---

## Related Documentation

- [Build System](build-system.md) - Gradle configuration
- [Quality Gates](quality-gates.md) - Code quality tools
- [Module Structure](module-structure.md) - Multi-module architecture
- [Domain Layer](domain-layer.md) - Pure Kotlin business logic
- [Data Layer](data-layer.md) - Android integrations
- [Presentation Layer](presentation-layer.md) - Jetpack Compose UI

---

**Back to**: [Android Implementation Overview](README.md)
