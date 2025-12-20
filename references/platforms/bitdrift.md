# bitdrift Capture Integration Guide

bitdrift Capture is a dynamic observability platform for mobile. Lightweight SDK with real-time control plane for selective data upload.

## Quick Start

### iOS (Swift)

```bash
# Swift Package Manager
.package(url: "https://github.com/bitdriftlabs/capture-ios", from: "0.21.0")

# CocoaPods
pod 'BitdriftCapture', '~> 0.21.0'
```

```swift
import Capture

func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {

    Capture.Logger.start(
        withAPIKey: "YOUR_API_KEY",
        sessionStrategy: .activityBased,
        configuration: .init()
    )

    return true
}
```

### Android (Kotlin)

```kotlin
// build.gradle.kts
plugins {
    id("io.bitdrift.capture") version "0.21.0"
}

dependencies {
    implementation("io.bitdrift:capture:0.21.0")
}
```

```kotlin
class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()

        Capture.Logger.start(
            apiKey = "YOUR_API_KEY",
            sessionStrategy = SessionStrategy.ActivityBased,
            configuration = Configuration(),
            context = this
        )
    }
}
```

### React Native

```bash
npm install @bitdrift/react-native-capture
```

```typescript
import { Capture } from '@bitdrift/react-native-capture';

Capture.start({
  apiKey: 'YOUR_API_KEY',
  sessionStrategy: 'activityBased',
});
```

---

## Key Features

### Logging

```swift
// iOS - Level-specific logging
Capture.Logger.logInfo("User started checkout", fields: ["cart_size": "\(cart.items.count)"])
Capture.Logger.logWarning("Retry attempt", fields: ["attempt": "2"])
Capture.Logger.logError("Payment failed", fields: ["error_code": error.code], error: error)

// Trace and debug for verbose logging
Capture.Logger.logTrace("Network request started")
Capture.Logger.logDebug("Cache hit for product \(productId)")
```

```kotlin
// Android - Level-specific logging
Capture.Logger.logInfo { "User started checkout" }
Capture.Logger.logWarning(fields = mapOf("attempt" to "2")) { "Retry attempt" }
Capture.Logger.logError(throwable = error, fields = mapOf("error_code" to error.code)) { "Payment failed" }
```

### Screen Tracking

```swift
// iOS
Capture.Logger.logScreenView(screenName: "CheckoutScreen")
```

```kotlin
// Android
Capture.Logger.logScreenView("CheckoutScreen")
```

### App Launch TTI

```swift
// iOS - Call when app is truly interactive
Capture.Logger.logAppLaunchTTI(duration)
```

```kotlin
// Android
Capture.Logger.logAppLaunchTTI(durationMs)
```

### Spans

```swift
// iOS - Track operations with timing
let span = Capture.Logger.startSpan(
    name: "checkout.process_payment",
    level: .info,
    fields: ["amount": "\(order.total)"]
)

// ... payment processing

span?.end()
```

```kotlin
// Android - Inline span tracking
Capture.Logger.trackSpan("checkout.process_payment") {
    // payment processing
    processPayment()
}

// Or manual span management
val span = Capture.Logger.startSpan("checkout.process_payment")
try {
    processPayment()
} finally {
    span?.end()
}
```

### Network Logging

```swift
// iOS - Log HTTP requests/responses
Capture.Logger.log(level: .debug, HTTPRequestInfo(
    method: "POST",
    url: url,
    headers: sanitizedHeaders
))

Capture.Logger.log(level: .debug, HTTPResponseInfo(
    statusCode: response.statusCode,
    duration: requestDuration
))
```

```kotlin
// Android - Automatic OkHttp instrumentation via plugin
// In build.gradle.kts:
bitdriftCapture {
    automaticOkHttpInstrumentation.set(true)
}
```

### Session Management

```swift
// iOS
let sessionId = Capture.Logger.sessionID
let sessionUrl = Capture.Logger.sessionURL  // Link to bitdrift dashboard

// Start fresh session (e.g., on logout)
Capture.Logger.startNewSession()
```

```kotlin
// Android
val sessionId = Capture.Logger.sessionId
val sessionUrl = Capture.Logger.sessionUrl

Capture.Logger.startNewSession()
```

### Device Identification

```swift
// iOS - Persistent device ID
let deviceId = Capture.Logger.deviceID
```

```kotlin
// Android
val deviceId = Capture.Logger.deviceId
```

### Global Fields

```swift
// iOS - Attach to all subsequent logs
Capture.Logger.addField(withKey: "user_tier", value: "premium")
Capture.Logger.addField(withKey: "app_version", value: Bundle.main.appVersion)

// Remove when no longer relevant
Capture.Logger.removeField(withKey: "user_tier")
```

```kotlin
// Android
Capture.Logger.addField("user_tier", "premium")
Capture.Logger.removeField("user_tier")
```

### Feature Flags

```swift
// iOS - Track feature flag exposure
Capture.Logger.setFeatureFlagExposure(withName: "checkout_v2", variant: "enabled")
```

```kotlin
// Android
Capture.Logger.setFeatureFlagExposure("checkout_v2", "enabled")
```

### Sleep Mode

```swift
// iOS - Reduce SDK activity (e.g., when app backgrounded)
Capture.Logger.setSleepMode(.full)
Capture.Logger.setSleepMode(.normal)
```

```kotlin
// Android
Capture.Logger.setSleepMode(SleepMode.FULL)
Capture.Logger.setSleepMode(SleepMode.NORMAL)
```

### Real-Time Streaming

```swift
// iOS - Generate code for live log streaming
Capture.Logger.createTemporaryDeviceCode { result in
    switch result {
    case .success(let code):
        print("Stream code: \(code)")
    case .failure(let error):
        print("Failed: \(error)")
    }
}
```

---

## Session Strategies

| Strategy | Behavior |
|----------|----------|
| `activityBased` | New session after app backgrounded for threshold |
| `fixed` | Session lasts fixed duration regardless of activity |

---

## Dashboard Features

### Timeline / Session Replay
- Visual timeline of user sessions
- All logs, screens, network calls in context
- Jump to specific moments

### Workflows
- Deploy monitoring rules without app release
- Adjust data collection levels remotely
- Target specific users or cohorts

### Instant Insights
- Auto-generated health dashboards
- Crash rates, error trends, performance metrics

### Alerts
- Configure alerts on any workflow or metric
- Slack, PagerDuty integrations

---

## Best Practices

| Practice | Implementation |
|----------|----------------|
| **Log app launch TTI** | Call `logAppLaunchTTI` when truly interactive |
| **Track all screens** | Use `logScreenView` on every screen appear |
| **Add global context** | Set user tier, app version via `addField` |
| **Use spans for operations** | Wrap async work in spans for timing |
| **Use appropriate levels** | Trace/Debug for verbose, Info for events, Error for failures |

---

## Anti-Patterns

| Don't | Why | Do Instead |
|-------|-----|------------|
| Log PII in fields | Compliance risk | Use anonymized IDs |
| Log high-cardinality values | Storage explosion | Use bucketed values |
| Skip screen tracking | Breaks timeline visualization | Track every screen |
| Ignore session URL | Lose debugging context | Log sessionURL on errors |

---

## Symbolication

bitdrift handles symbolication via the Gradle plugin for Android:

```kotlin
// build.gradle.kts
bitdriftCapture {
    uploadProguardMappings.set(true)
}
```

For iOS, dSYM upload is configured via the bitdrift dashboard or CLI.

---

## Links

- [bitdrift Docs](https://docs.bitdrift.io)
- [SDK Quickstart](https://docs.bitdrift.io/sdk/quickstart.html)
- [GitHub - capture-sdk](https://github.com/bitdriftlabs/capture-sdk)
- [GitHub - capture-ios](https://github.com/bitdriftlabs/capture-ios)
- [iOS Releases](https://docs.bitdrift.io/sdk/releases-ios.html)
- [Android Releases](https://docs.bitdrift.io/sdk/releases-android.html)
