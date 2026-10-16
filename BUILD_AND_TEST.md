# Build & Test Guide – RemoteHotspot Controller Pro

## 1. Project Layout (delivered)

```
RemoteHotspotControllerPro/
├── README.md
├── BUILD_AND_TEST.md
├── common/
│   └── src/main/java/com/remotehotspot/common/
│       ├── protocol/BluetoothProtocol.kt
│       ├── model/DeviceModels.kt
│       └── bluetooth/BluetoothConnectionManager.kt
├── admin-app/
│   ├── build.gradle.kts
│   └── src/main/
│       ├── AndroidManifest.xml
│       └── java/com/remotehotspot/admin/
│           ├── data/DeviceRepository.kt
│           ├── viewmodel/AdminViewModel.kt
│           └── ui/main/AdminMainActivity.kt   (Compose Material 3 device list)
│           └── ui/theme/Theme.kt
└── user-app/
    ├── build.gradle.kts
    └── src/main/
        ├── AndroidManifest.xml
        ├── res/xml/accessibility_service_config.xml
        ├── res/values/strings.xml
        └── java/com/remotehotspot/user/
            ├── accessibility/
            │   ├── HotspotAccessibilityService.kt   (primary)
            │   └── HotspotAccessibilityServiceJava.java
            ├── service/
            │   ├── UserForegroundService.kt
            │   └── BootReceiver.kt
            ├── util/OemSurvivalHelper.kt
            └── ui/
                ├── main/UserMainActivity.kt
                ├── setup/SetupWizardActivity.kt
                └── theme/Theme.kt
```

## 2. How to open in Android Studio

1. Create a new empty Android project (or two modules).
2. Copy `admin-app` sources into one module, `user-app` into the other.
3. Copy the entire `common/src/main/java/...` tree into **both** modules (or convert `common` into a proper Android library module and depend on it).
4. Add the Gradle plugins and dependencies from the provided `build.gradle.kts` files.
5. Create placeholder mipmap icons (`ic_launcher`) or use the default Android Studio ones.
6. Sync Gradle.

### Required root `settings.gradle.kts` (example multi-module)

```kotlin
rootProject.name = "RemoteHotspotControllerPro"
include(":admin-app")
include(":user-app")
include(":common")
```

## 3. Critical reliability checklist before testing

### User device
- [ ] Accessibility Service enabled (Setup Wizard step 1)
- [ ] Battery optimization ignored (step 2)
- [ ] OEM auto-start enabled for Realme / Samsung / Infinix (step 3)
- [ ] Bluetooth ON
- [ ] Notification permission granted (Android 13+)
- [ ] Screen is ON when sending Hotspot commands

### Admin device
- [ ] Bluetooth ON + permissions granted
- [ ] Location permission if API ≤ 30 (for discovery)

## 4. Test sequence

1. Install **User App** → complete Setup Wizard fully.
2. Confirm foreground notification “Listening for Admin…”.
3. Install **Admin App** → grant Bluetooth permissions.
4. Tap “+” → Scan → select the User device → Connect.
5. Device card appears with Online status.
6. While User screen is **ON**, tap ON / OFF / Toggle.
7. Observe Quick Settings open and Hotspot tile change.
8. Force-stop User app → wait a few seconds → service should restart (BootReceiver / AlarmManager).
9. Lock User screen → send command → should receive `SCREEN_OFF` status.
10. Test on Realme, Samsung, Infinix for OEM-specific survival.

## 5. Known limitations & production hardening

- Hotspot state reporting is best-effort until after a successful toggle verification.
- Coordinate calibration UI is not yet shipped (API exists: `saveCalibration(x,y)`).
- Pairing currently uses simple connect; add PIN confirmation dialog for production.
- No encryption on RFCOMM; for production wrap with TLS or use encrypted Bluetooth sockets.
- Icons, about page, log export UI are left as extension points.

## 6. Extending

- Add a Settings screen in Admin for theme, auto-reconnect interval, notification toggles.
- Add coordinate calibration activity in User that opens QS and captures a touch.
- Add Crashlytics / Sentry stubs in `Application` class.
- Convert `common` into a true shared library module.
