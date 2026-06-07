# TruckLink Driver

**[View UI & project showcase → TruckLink on portfolio](https://bassam-jawish-portfolio.vercel.app/projects/trucklink)**

Cross-platform Flutter application for truck drivers to join trips via a trip code, navigate checkpoints on a live map, update trip status, stream location to the backend, and manage shipment orders. The app targets **Android** and **iOS** and communicates with a REST backend.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | Flutter (Dart SDK `^3.8.1`) |
| State management & routing | [GetX](https://pub.dev/packages/get) |
| HTTP client | [Dio](https://pub.dev/packages/dio) + `pretty_dio_logger` (debug) |
| Dependency injection | [GetIt](https://pub.dev/packages/get_it) + GetX service locator |
| Local storage | `flutter_secure_storage` (`SecureStorage`) |
| Real-time updates | [Pusher Channels](https://pub.dev/packages/pusher_channels_flutter) |
| Maps & location | `google_maps_flutter`, `geolocator`, `permission_handler` |
| Deep linking | `app_links` |
| UI utilities | `flutter_screenutil`, `cached_network_image`, `lottie`, `flutter_svg`, `flutter_spinkit` |
| Localization | Flutter gen-l10n (English & Arabic) |
| Asset codegen | [flutter_gen](https://pub.dev/packages/flutter_gen) |
| API config | `lib/config/config.dart` (`Configuration` class) |

---

## Architecture

The codebase follows a **feature-first modular layout**. Each feature lives under `lib/modules/` and is organized into predictable layers:

```
module/
├── controller/   # GetX controllers — UI state, user actions
├── service/        # API calls and business logic
├── model/          # Data models and API response types
└── view/
    ├── pages/      # Screen widgets
    └── components/ # Feature-specific reusable widgets
```

Shared infrastructure sits in `lib/core/` (networking, storage, widgets, enums, error handling) and cross-cutting app configuration in `lib/config/` (theme, router, API base URL, device info).

### Data flow

```
View (Page/Widget)
    ↕  GetX reactive bindings
Controller
    ↕  async calls
Service  →  ApiService (Dio)  →  REST API
    ↕
Model (parsed response)
```

### App bootstrap

1. `main.dart` — initializes Flutter bindings, locks portrait orientation, and calls `AppBindings.init()`.
2. `lib/config/bindings.dart` — initializes secure storage, DI container, locale/theme preferences, and deep links.
3. `lib/injection_container.dart` — registers global services (`ApiService`, `AuthService`, `HomeService`, `OrderService`, `PermissionManager`).
4. `main.dart` (`MyApp`) — builds `GetMaterialApp` with routes, themes, and localization.
5. `SplashPage` — checks for a stored trip token and routes to home or login.

Route-level controllers are registered lazily via GetX `BindingsBuilder` in `lib/config/router/navigation_manager.dart`.

---

## Project Structure

```
trucklink_driver/
├── android/                  # Android native project (Kotlin, Gradle KTS)
├── ios/                      # iOS native project
├── assets/
│   ├── animations/           # Lottie JSON files
│   ├── icons/                # SVG map markers
│   └── images/
│       ├── logo/             # TruckLink brand assets
│       └── placeholders/
├── lib/
│   ├── main.dart             # Entry point & GetMaterialApp
│   ├── injection_container.dart
│   ├── firebase_api.dart     # Firebase/FCM stub (currently commented out)
│   ├── auto_generated/       # flutter_gen output (assets)
│   ├── config/
│   │   ├── bindings.dart     # App bootstrap
│   │   ├── config.dart       # API base URL & storage URL
│   │   ├── constants.dart
│   │   ├── endpoints.dart
│   │   ├── device_utils.dart
│   │   ├── router/           # Routes & NavigationManager
│   │   └── theme/            # Light/dark themes, colors, styles
│   ├── core/
│   │   ├── enums/            # Loading states, etc.
│   │   ├── error/            # Exceptions, failures, error codes
│   │   ├── middleware/       # AuthGuard, AuthHelper
│   │   ├── network/          # ApiService, network info, failures
│   │   ├── repos/            # Shared repositories
│   │   ├── utils/            # SecureStorage, Pusher, deep links, validators
│   │   └── widgets/          # Shared UI components
│   ├── l10n/                 # ARB files & generated localizations
│   └── modules/
│       ├── auth/             # Splash, trip-code login, deep link parsing
│       ├── home/             # Map, trip details, checkpoints, live tracking
│       ├── orders/           # Shipment orders, filters, pagination
│       └── settings/         # Theme, language, preferences, help & feedback
├── pubspec.yaml
├── l10n.yaml
└── android/app/proguard-rules.pro  # Secure storage crypto keep rules (release)
```

---

## Features & Modules

| Module | Responsibility |
|---|---|
| **auth** | Trip-code login (code stored as Bearer token), splash auth redirect, deep-link trip code prefill |
| **home** | Google Maps with checkpoint markers, trip status workflow, GPS location sync, Pusher real-time channel |
| **orders** | Paginated shipment list, city/status filters, order statistics tied to active trip |
| **settings** | Language (EN/AR), theme mode (light/dark/system), notification & location toggles |
| **settings (help)** | FAQ, contact info, feedback form UI |

The primary shell is `HomePage` with a **drawer** for navigation to Orders, Settings, Help & Feedback, and Logout. There is no bottom-tab shell.

### Trip workflow (home)

Drivers progress through checkpoint-based actions (arrive at factory, start trip, arrive at receiver, checkpoint transitions, delivery) driven by `HomePageController`. Status and GPS coordinates are posted to the backend; live updates are received over a private Pusher channel.

---

## Prerequisites

- [Flutter SDK](https://docs.flutter.dev/get-started/install) compatible with Dart `^3.8.1`
- Android Studio / Xcode for platform builds
- Google Maps API keys for Android and iOS (map rendering & directions)
- Access to the TruckLink backend API
- Pusher app credentials (configured in `lib/core/utils/pusher_service.dart`)

Verify your setup:

```bash
flutter doctor
```

---

## API Configuration

Update `lib/config/config.dart` with your backend URLs:

```dart
class Configuration {
  static const String baseUrl = "https://your-api-host.com/api/";
  static const String storageUrl = "https://your-api-host.com/storage/";
}
```

| Constant | Description |
|---|---|
| `baseUrl` | REST API prefix used by `ApiService` |
| `storageUrl` | Base URL for uploaded media/assets |

Authentication uses the **trip code as the Bearer token** — there is no separate JWT refresh flow. On `401`/`403`, the token is cleared and the user is redirected to login.

---

## Getting Started

```bash
# Install dependencies
flutter pub get

# Generate type-safe asset references
dart run build_runner build --delete-conflicting-outputs

# Generate localizations (also runs via flutter generate)
flutter gen-l10n

# Run on a connected device or emulator
flutter run
```

### Build for release

```bash
flutter build apk --release       # Android APK
flutter build appbundle --release # Android App Bundle
flutter build ios --release       # iOS (requires macOS + Xcode)
```

---

## Networking

- All HTTP traffic goes through `ApiService` (`lib/core/network/remote_api_service.dart`).
- Request headers include `Content-Type`, `Accept`, `X-Device-ID`, and `Accept-Language`.
- Authenticated requests attach `Authorization: Bearer <tripCode>`.
- Responses are normalized via `ErrorHandler` into `AppResponse<T>` (`lib/core/utils/base_api/base_model.dart`).
- Debug builds log requests/responses via `PrettyDioLogger`.

### Key API endpoints

| Area | Method | Path |
|---|---|---|
| Auth / trip | `GET` | `driver/trip` |
| Trip status | `POST` | `driver/trip/update-status` |
| Location sync | `POST` | `driver/trip/update-location` |
| Shipments | `GET` | `driver/trip/shipments` |
| Trip cities | `GET` | `trips/:tripId/cities` |
| Order detail | `GET` | `orders/:orderId` |
| Pusher auth | `POST` | `custom-broadcasting/auth` |

---

## Real-Time (Pusher)

`PusherService` (`lib/core/utils/pusher_service.dart`) manages the WebSocket connection. After trip details load, `HomePageController` subscribes to a private channel (`private-private-trip.<tripCode>`) and listens for events such as `location.updated` and `status.updated`.

Channel authorization is delegated to the backend via `custom-broadcasting/auth` using the stored trip token.

---

## Maps & Location

`MapComponent` (`lib/modules/home/view/components/map_component.dart`) wraps `google_maps_flutter` with:

- Checkpoint markers (custom SVG assets)
- Camera navigation to checkpoints and current position
- Continuous location tracking via `geolocator`
- Periodic location posts through `HomeService.updateLocation`

Location permissions are handled by `PermissionManager` in `lib/core/utils/permission_manager.dart`.

---

## Deep Linking

`DeepLinkService` (`lib/core/utils/deep_link_service.dart`) uses `app_links` to capture incoming URIs on cold start and while the app is running. `AuthService.parseTripCodeFromLink` extracts a trip code from:

- Query parameter `?tripCode=...`
- `trucklink.com/<code>` path segments

Pending codes are consumed on the login screen to prefill the trip code field.

---

## Localization

Supported locales: **English (`en`)** and **Arabic (`ar`)**.

- Source strings: `lib/l10n/app_en.arb`, `lib/l10n/app_ar.arb`
- Generated classes: `lib/l10n/app_localizations.dart`
- Locale list: `lib/l10n/l10n.dart`

Language and theme preferences are managed by `SettingsController` and persisted in `SecureStorage`.

---

## Code Generation

The project uses **flutter_gen** for type-safe asset access:

```bash
dart run build_runner build --delete-conflicting-outputs
```

Generated output: `lib/auto_generated/assets.gen.dart`

Usage example:

```dart
import 'auto_generated/assets.gen.dart';

Assets.images.logo.greenIcon.image();
Assets.icons.selectedMarker.svg();
```

App launcher icons are configured in `pubspec.yaml` under `flutter_launcher_icons`.

---

## Storage

| Store | Usage |
|---|---|
| `SecureStorage` + `flutter_secure_storage` | Trip auth token, language code, theme mode, settings flags |

`SecureStorage` includes health checks and corruption recovery (relevant for Android release builds — see `android/app/proguard-rules.pro`).

---

## Navigation

Routes are declared in `AppRoutes` and registered in `NavigationManager.getPages`:

| Route | Path |
|---|---|
| Splash | `/` |
| Login | `/login` |
| Home (map shell) | `/home` |
| Orders | `/orders` |
| Settings | `/settings` |
| Help & Feedback | `/help-feedback` |

`AuthHelper` (`lib/core/middleware/auth_guard.dart`) provides async token checks and logout helpers used across modules.

---

## Testing

```bash
flutter test
```

No widget tests are included yet. Add tests under a `test/` directory as needed.

---

## License

Private project — not published to pub.dev (`publish_to: 'none'`).
