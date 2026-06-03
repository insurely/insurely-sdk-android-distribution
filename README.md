# Insurely Android SDK

The Insurely Android SDK lets you embed the Insurely user interface in your Android app. The SDK loads the Insurely web experience in a `WebView` and exposes a Jetpack Compose-based UI surface plus a small set of configuration, prefill, and event APIs.

## Requirements

- Android API level 24 (Android 7.0) or later
- Jetpack Compose
- Kotlin 1.9 or later

Access to this repository is granted to authorized Insurely customers. Use of the SDK is governed by the terms of your agreement with Insurely AB. See `LICENSE` for details.

## Access

This SDK is distributed via a private GitHub repository. Every developer who needs to install or update the SDK requires read access. Send the GitHub usernames of every developer who will integrate the SDK to your Insurely account representative, and we will grant each account read access to this repository.

How long that access is needed depends on the installation method you choose — see [Installation](#installation) for the specific pattern each method uses.

## Installation

The SDK can be installed two ways. Both produce an identical AAR in your app — they differ only in how the framework reaches your project. Pick the one that fits your team's environment.

### Vendored clone (recommended)

Best when you want to vendor the SDK in your repository for reproducibility, build in air-gapped CI, or pin to a specific commit. This is also the simplest path if you already have a working multi-module Gradle build.

1. Clone this repository to a stable location in or alongside your project:
   ```
   git clone git@github.com:insurely/insurely-sdk-android-distribution.git
   ```
2. In your project's `settings.gradle.kts`, add the cloned directory as a `flatDir` repository:
   ```kotlin
   dependencyResolutionManagement {
       repositories {
           google()
           mavenCentral()
           flatDir {
               dirs("path/to/insurely-sdk-android-distribution")
           }
       }
   }
   ```
3. In your app module's `build.gradle.kts`, reference the AAR:
   ```kotlin
   dependencies {
       implementation(files("path/to/insurely-sdk-android-distribution/InsurelySDK-release.aar"))
       // Or, if you want a separate debug build:
       // debugImplementation(files("path/to/insurely-sdk-android-distribution/InsurelySDK-debug.aar"))
       // releaseImplementation(files("path/to/insurely-sdk-android-distribution/InsurelySDK-release.aar"))
   }
   ```
4. Sync Gradle. The SDK is available under `import com.insurely.blocks.sdk.*`.

To update, run `git pull` in the clone (or `git checkout <version-tag>` to pin to an exact version) and rebuild your app.

No CI deploy key is required — the SDK is vendored alongside your own code, so CI never reaches out to this repository at build time.

### Direct AAR download

Best when you do not maintain a clone of this repository and prefer to drop a versioned binary into your project directly.

1. Download `InsurelySDK-release.aar` (and optionally `InsurelySDK-debug.aar`) from the [latest GitHub Release](https://github.com/insurely/insurely-sdk-android-distribution/releases).
2. Place the AAR(s) into your app module's `libs/` directory.
3. In your project's `settings.gradle.kts`, add a `flatDir` repository pointing at `libs/`:
   ```kotlin
   dependencyResolutionManagement {
       repositories {
           google()
           mavenCentral()
           flatDir {
               dirs("libs")
           }
       }
   }
   ```
4. In your app module's `build.gradle.kts`:
   ```kotlin
   dependencies {
       implementation(files("libs/InsurelySDK-release.aar"))
   }
   ```

Version tracking and update notifications are entirely manual with this method. No CI deploy key is required.

## Required dependencies

The SDK is distributed as a standalone AAR. Because AAR plus `flatDir` does not resolve transitive dependencies the way Maven coordinates do, your app must declare the libraries the SDK depends on explicitly. Add these to your app module's `dependencies { }` block alongside the SDK reference:

```kotlin
// Required transitive dependencies of InsurelySDK
implementation("org.jetbrains.kotlinx:kotlinx-serialization-json:1.6.3")
implementation("io.ktor:ktor-client-core:2.3.10")
implementation("io.ktor:ktor-client-cio:2.3.10")
implementation("io.ktor:ktor-client-logging:2.3.10")
implementation("io.ktor:ktor-client-content-negotiation:2.3.10")
implementation("io.ktor:ktor-serialization-kotlinx-json:2.3.10")
```

Pin the exact versions shown above for the SDK version you are integrating. Without these dependencies, the SDK compiles successfully but crashes at runtime with `NoClassDefFoundError` when the embedded `WebView` is first composed.

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

The SDK follows [Semantic Versioning](https://semver.org/). Release notes for each version are published on the [Releases page](https://github.com/insurely/insurely-sdk-android-distribution/releases).

For now, the SDK is distributed as a vendored AAR rather than via Maven coordinates. New releases require a manual pull of this repository (or a fresh download from the Releases page) to update your local copy.

## Support

For integration questions, bug reports, or feature requests, contact your Insurely account representative.
