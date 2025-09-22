# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Build and Development

```bash
# Install dependencies
flutter pub get

# Run the app on connected device/emulator
flutter run

# Build for Android
flutter build apk
flutter build appbundle  # For Play Store

# Build for iOS
flutter build ios

# Clean build artifacts
flutter clean

# Analyze code for issues
flutter analyze

# Format code
dart format lib/
```

### Localization

```bash
# Generate localization files (after modifying .arb files)
flutter gen-l10n
```

## Architecture

### Core Application Structure

The app is a Flutter-based GPS tracking client that communicates with Traccar servers. It runs continuously in the background to send location updates.

**Key Services:**

1. **GeolocationService** (`lib/geolocation_service.dart`):
   - Core background location tracking using `flutter_background_geolocation` plugin
   - Implements intelligent location filtering based on movement, distance, interval, and angle parameters
   - Headless task support for Android to handle location updates when app is terminated
   - Location caching and sync with server

2. **ConfigurationService** (`lib/configuration_service.dart`):
   - Handles deep link and URI-based configuration
   - Updates tracking parameters dynamically from URLs (e.g., `https://server.com?interval=30&distance=50`)

3. **Preferences** (`lib/preferences.dart`):
   - Centralized configuration management using SharedPreferences
   - Manages tracking parameters: accuracy, distance, interval, angle, heartbeat, buffer settings
   - Provides geolocation configuration for background plugin

4. **PushService** (`lib/push_service.dart`):
   - Firebase Cloud Messaging integration for remote control
   - Handles push notifications and remote commands

### Platform-Specific Implementation

**Android** (`android/app/build.gradle.kts`):
- Uses Kotlin DSL for Gradle configuration
- Integrates Firebase services (Analytics, Crashlytics, Messaging)
- Background geolocation plugin configuration
- Signing configuration references external key.properties file

**iOS** (`ios/`):
- Background location modes enabled
- Push notification capabilities
- App Store configuration

### Location Tracking Algorithm

The app uses sophisticated filtering in `GeolocationService._shouldDelete()` to optimize battery usage:

1. Keeps all stationary locations
2. Filters moving locations based on:
   - Distance from last location (`distance` parameter)
   - Time since last update (`interval` and `fastestInterval`)
   - Heading change (`angle` parameter for highest accuracy mode)
3. Prioritizes heartbeat and extra-tagged locations

### Server Communication

- Sends location data to configured Traccar server URL
- Format: `POST` to server with device ID and location data
- Supports buffering locations when offline (`buffer` preference)
- Auto-sync when connection restored

### Key Dependencies

- `flutter_background_geolocation`: Core location tracking
- `firebase_*`: Analytics, crash reporting, push notifications
- `shared_preferences`: Configuration storage
- `app_links`: Deep linking support
- `mobile_scanner`: QR code scanning for configuration
- `rate_my_app`: App store rating prompts