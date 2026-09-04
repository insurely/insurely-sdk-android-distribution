# Insurely Android SDK

The Insurely Android SDK lets you embed the Insurely user interface in your Android app. The SDK loads the Insurely web experience in a `WebView` and exposes a Jetpack Compose-based UI surface plus a small set of configuration, prefill, and event APIs.

## Requirements

- Android API level 24 (Android 7.0) or later
- Jetpack Compose
- Kotlin 1.9 or later

Use of the SDK is governed by the terms of your agreement with Insurely AB. See `LICENSE` for details.

## Access

This repository is public. No access request, GitHub account, or credential is
needed to install the SDK or to update it — from a developer machine or from
CI.

The SDK itself remains proprietary: it is published so you can install, inspect
and audit it, under the terms of your agreement with Insurely AB. See `LICENSE`.

## Installation

```kotlin
dependencies {
    implementation("com.insurely:insurely-android-sdk:1.2.2")
}
```

That is the whole installation. `mavenCentral()` is already in most projects'
repository list; if it is not, add it in `settings.gradle.kts`:

```kotlin
dependencyResolutionManagement {
    repositories {
        google()
        mavenCentral()
    }
}
```

Sync Gradle. The SDK is available under `import com.insurely.blocks.sdk.*`.

Gradle resolves the SDK's own dependencies from its POM, so there is nothing
else to declare — see [Upgrading from the AAR](#upgrading-from-the-aar) if you
integrated before 1.2.2.

### Air-gapped builds

If your build cannot reach Maven Central, every release also attaches
`InsurelySDK-release.aar` to its [GitHub Release](https://github.com/insurely/insurely-sdk-android-distribution/releases),
and this repository holds the latest at the root.

Consuming the AAR directly means Gradle resolves no transitive dependencies
for it, so your app has to declare them itself:

```kotlin
dependencies {
    implementation(files("path/to/InsurelySDK-release.aar"))

    // Required only when consuming the AAR directly. The Maven coordinate
    // above carries these for you.
    implementation("org.jetbrains.kotlinx:kotlinx-serialization-json:1.6.3")
    implementation("io.ktor:ktor-client-core:2.3.10")
    implementation("io.ktor:ktor-client-cio:2.3.10")
    implementation("io.ktor:ktor-client-logging:2.3.10")
    implementation("io.ktor:ktor-client-content-negotiation:2.3.10")
    implementation("io.ktor:ktor-serialization-kotlinx-json:2.3.10")
}
```

Pin the exact versions shown, for the SDK version you are integrating. Without
them the SDK compiles successfully and then crashes at runtime with
`NoClassDefFoundError` when the embedded `WebView` is first composed.

## Upgrading from the AAR

If you integrated before 1.2.2, switching to the Maven coordinate lets you
delete three things:

1. the vendored AAR or downloaded file, and any `flatDir` repository entry
2. the `implementation(files("…"))` line
3. **all six** hand-declared `ktor` and `kotlinx-serialization` dependencies —
   the POM now carries them, at the same versions

Nothing else changes. The binary is the same one you have today.

## Quickstart

The SDK is reached through a Jetpack Compose composable, `InsurelyView`. A minimal integration:

```kotlin
import androidx.compose.runtime.Composable
import com.insurely.blocks.sdk.InsurelyView
import com.insurely.blocks.sdk.config.InsurelyConfig
import com.insurely.blocks.sdk.config.InsurelyEnvironment
import com.insurely.blocks.sdk.config.InsurelySettings

@Composable
fun InsurelyScreen() {
    InsurelyView(
        settings = InsurelySettings(
            environment = InsurelyEnvironment.Prod,
            config = InsurelyConfig(
                customerId = "your-customer-id",
                configName = "your-config-name",
            ),
        ),
        onResultsReceived = { results ->
            // Handle the collected data when the flow completes.
        },
        onEventReceived = { event ->
            // Handle SDK events (collection status, page views, etc.).
        },
        onErrorReceived = { error ->
            // Handle SDK errors.
        },
    )
}
```

Insurely provides your `customerId` and `configName` during onboarding. Use `InsurelyEnvironment.Test` while developing against the test environment.

## Theme mode

The SDK supports light, dark, and system-driven theming via the optional `themeMode` parameter on `InsurelyConfig`:

```kotlin
import com.insurely.blocks.sdk.config.InsurelyConfig
import com.insurely.blocks.sdk.config.InsurelyThemeMode

InsurelyConfig(
    customerId = "...",
    configName = "...",
    themeMode = InsurelyThemeMode.System,
)
```

- `InsurelyThemeMode.Light` or `InsurelyThemeMode.Dark` — render the corresponding theme variant unconditionally.
- `InsurelyThemeMode.System` — follow the host's color scheme via `prefers-color-scheme`. The embedded `WebView` already inherits the system color scheme on Android 10 and later, so this typically does the right thing without further configuration.
- Omitted (default `null`) — no preference is sent; blocks resolves the theme using its own defaults.

Dark mode rendering requires your `BlocksConfig` to include at least one `ConnectedTheme`. With both `light` and `dark` identifiers configured server-side, passing `themeMode = InsurelyThemeMode.System` follows the device automatically. Contact your Insurely account representative if you need help setting up `ConnectedTheme` rows for your configuration.

## Versioning

The SDK follows [Semantic Versioning](https://semver.org/). Release notes for
each version are published on the
[Releases page](https://github.com/insurely/insurely-sdk-android-distribution/releases).

### Staying up to date

Since 1.2.2 the SDK is a normal Gradle dependency, so your existing tooling
handles updates: version catalogs resolve it, and Dependabot or Renovate will
open a pull request when a release ships. Nothing here needs watching.

If you consume the AAR directly, that does not apply — Gradle never learns a
version exists. Download the new AAR from the Releases page and replace the
file, and use **Watch → Custom → Releases** at the top of this repository so
you hear about one.

## Support

For integration questions, bug reports, or feature requests, contact your Insurely account representative.
