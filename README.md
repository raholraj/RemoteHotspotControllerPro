# RemoteHotspot Controller Pro

**Dual-app offline Bluetooth system for remote Mobile Hotspot control.**

- **Admin App** (`com.remotehotspot.admin`): Discover, pair, monitor & control multiple User devices.
- **User App** (`com.remotehotspot.user`): Runs as persistent background service + Accessibility Service to toggle Hotspot reliably.

## Requirements
- Min SDK: 26 (Android 8.0)
- Target SDK: 35 (Android 15)
- Language: Kotlin (primary) + Java equivalents for critical classes
- UI: Jetpack Compose + Material 3
- Communication: Bluetooth Classic (RFCOMM) with heartbeat

## Extreme Reliability Features
- Multi-layer Hotspot toggle via Accessibility Service (text match → content-desc → coordinate calibration → Settings fallback)
- Only acts when screen is ON
- Persistent foreground service + BootReceiver
- OEM-specific guidance (Realme UI, Samsung One UI, Infinix XOS)
- Ignore battery optimizations + auto-start prompts
- Heartbeat every 30s + exponential backoff reconnect
- Graceful multi-device handling

## Project Structure
```
RemoteHotspotControllerPro/
├── common/                  # Shared protocol, models, Bluetooth helpers
├── admin-app/               # Admin application
├── user-app/                # User / controllable device application
└── README.md
```

## How to Build

### Option A – Android Studio (recommended)
1. Open Android Studio → New Project → Empty Activity (or Import).
2. Create two modules or two separate projects pointing to `admin-app` and `user-app`.
3. Copy the `common` sources into both (or create a shared library module).
4. Sync Gradle, resolve dependencies.
5. Build & run on real devices (Bluetooth + Accessibility require physical hardware).

### Option B – Command line
```bash
# From each app directory after placing proper build.gradle files
./gradlew assembleDebug
```

## Testing Checklist
1. Install **User App** first → complete setup wizard (Accessibility, Battery, Auto-start).
2. Install **Admin App** → enable Bluetooth → scan & pair with User device (PIN shown on User).
3. Verify device card appears with battery, hotspot status, online indicator.
4. Toggle Hotspot ON/OFF from Admin while User screen is ON.
5. Kill User app process → confirm auto-recovery via BootReceiver / Service restart.
6. Test on Realme, Samsung, Infinix for OEM battery restrictions.

## Critical Classes
| Class | Location | Purpose |
|-------|----------|---------|
| `HotspotAccessibilityService` | user-app | Multi-layer Hotspot toggle |
| `BluetoothProtocol` | common | Command definitions + serialization |
| `BluetoothConnectionManager` | both | RFCOMM server/client + heartbeat |
| `OemSurvivalHelper` | user-app | Realme/Samsung/Infinix guidance |
| `DeviceCard` + Adapter | admin-app | Live status UI |

## Security Note
Simple shared secret / PIN pairing is used. For production consider certificate-based auth or encrypted RFCOMM.
