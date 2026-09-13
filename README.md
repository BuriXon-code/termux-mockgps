# termux-mockgps

`termux-mockgps` is a lightweight POSIX shell command for controlling the Android **[MockGPS](https://github.com/BuriXon-code/MockGPS)** application directly from Termux.

It is intentionally small: there is no local database, no configuration framework, and **no additional Termux package dependencies**. The command uses the Android `am` command to communicate with MockGPS.

## Relationship with MockGPS

This project is the command-line frontend for the Android application:

**[BuriXon-code/MockGPS](https://github.com/BuriXon-code/MockGPS)**

The two projects are designed to work together.

`termux-mockgps` sends commands to the exported MockGPS broadcast receiver and, where appropriate, starts or stops the Android foreground service.

MockGPS contains the actual location-mocking implementation and is responsible for storing the mock coordinates and managing Android's test providers.

## Features

- Start MockGPS from Termux.
- Start at the last saved location or specify coordinates.
- Change mock coordinates while MockGPS is running.
- Stop mocking and restore normal location operation.
- Enable or disable MockGPS boot restoration.
- Enable or disable location drift.
- Open the MockGPS Android application.
- Optional toast suppression with `-notoast`.
- POSIX `sh` implementation.
- No additional Termux package dependencies.

## Compatibility

`termux-mockgps` is designed specifically for the current **BuriXon-code/MockGPS** Android application.

It is intended for:

- **Termux** on Android.
- **Android 10 or newer** for the current MockGPS setup.
- The matching **MockGPS** Android application installed.
- MockGPS selected as the system mock-location application.

The script itself is portable POSIX `sh`, but it depends on Android's `am` command and the MockGPS application interface, so it is not a generic desktop Linux location-mocking command.

## Dependencies

### Termux

There are **no additional Termux package dependencies**.

You do not need to install packages such as `python`, `termux-api` or whatever.

The script uses:

- POSIX shell functionality provided by the Termux environment.
- Android `am` for starting services and sending broadcasts.
- Android `pm` to check whether MockGPS is installed.
> [!NOTE]  
> The script uses Android pm commands because the ones in Termux don't work properly.

The script automatically looks for `am` in the normal Termux/Android locations.

## Installation

Clone the repository:

```sh
git clone https://github.com/BuriXon-code/termux-mockgps.git
cd termux-mockgps
```

Copy the command into Termux's executable directory:

```sh
cp termux-mockgps "$PREFIX/bin/termux-mockgps"
chmod +x "$PREFIX/bin/termux-mockgps"
```

Verify it:

```sh
termux-mockgps help
```

You can also run the script directly from the repository:

```sh
./termux-mockgps help
```

## Android preparation

Before using the command, configure the companion MockGPS application.

### Grant location permission

Open:

```text
Settings
→ Apps
→ MockGPS
→ Permissions
→ Location
```

Grant location access.

### Allow notifications

Open:

```text
Settings
→ Apps
→ MockGPS
→ Notifications
```

Enable notifications.

### Select MockGPS as the mock-location application

Open:

```text
Settings
→ Developer options
→ Select mock location app
```

Select:

```text
MockGPS
```

Without this configuration Android will reject the injected test locations.

## Usage

```text
usage: termux-mockgps <command> [options]

Commands:
  start [lat lon]       Start mocking a location.
  set <lat lon>         Set the mock location.
  boot <0|1|on|off>     Enable or disable mocking on device boot.
  drift <0|1|on|off>    Enable or disable location drift.
  open                  Open the MockGPS application.
  stop                  Stop mocking and restore normal location.
  help                  Show this help message.
  version               Show version information.

Location:
  Coordinates can be specified as:
    50.06143 19.93658
    50.06143,19.93658
    -lat:50.06143 -lon:19.93658

Boot:
  Values:
    1|on                Enable mocking on boot.
    0|off               Disable mocking on boot.

Options:
  -notoast              Do not display toast notification.
```

## Start

Start MockGPS using its last stored location:

```sh
termux-mockgps start
```

Start at a specific location:

```sh
termux-mockgps start 50.06143 19.93658
```

Negative coordinates are supported:

```sh
termux-mockgps start -50.06143 -19.93658
```

Explicit coordinate options are supported too:

```sh
termux-mockgps start -lat:50.06143 -lon:19.93658
```

## Set

Change the mock location:

```sh
termux-mockgps set 50.06143 19.93658
```

Comma-separated coordinates are also supported:

```sh
termux-mockgps set 50.06143,19.93658
```

Or use explicit options:

```sh
termux-mockgps set -lat:50.06143 -lon:-19.93658
```

When MockGPS is running, the new coordinates are applied automatically.

When MockGPS is stopped, the new coordinates are stored by the Android application and used by the next `start` command.

Suppress the command-related toast:

```sh
termux-mockgps set 50.06143 19.93658 -notoast
```

## Stop

Stop the MockGPS foreground service and disable persistent operation:

```sh
termux-mockgps stop
```

The Android application remains installed and keeps its stored coordinates.

## Boot

Enable restoration after Android boot:

```sh
termux-mockgps boot on
```

The following forms are also accepted:

```sh
termux-mockgps boot 1
termux-mockgps boot enable
termux-mockgps boot true
```

Disable it:

```sh
termux-mockgps boot off
```

or:

```sh
termux-mockgps boot 0
```

Boot restoration itself is implemented by MockGPS. The Termux command only changes the corresponding application state through the broadcast interface.

## Drift

Enable small simulated movement around the selected location:

```sh
termux-mockgps drift on
```

Disable it:

```sh
termux-mockgps drift off
```

Other accepted forms include `1`, `0`, `enable`, `disable`, `true`, and `false`.

The drift implementation belongs to the MockGPS Android service.

## Open

Open the MockGPS Android application:

```sh
termux-mockgps open
```

This uses the application's launcher activity.

> [!NOTE]  
> The -notoast option has no effect on start/stop commands because they do not send commands via droadcast.

## Version

Show the installed script version:

```sh
termux-mockgps version
```

Current script version:

```text
1.0.0
```

## How communication works

The important part of the project is its integration with MockGPS through Android's command interface.

Conceptually:

```text
Termux
  │
  ├── start/stop service commands
  │
  └── broadcasts
        │
        ▼
MockGPS / MockGpsReceiver
        │
        ▼
MockLocationService
        │
        ▼
Android test location providers
```

The companion script uses the MockGPS package:

```text
dev.burixon.mockgps
```

and the Android components:

```text
.MockLocationService
.MockGpsReceiver
```

This is why `termux-mockgps` is intended specifically for the matching MockGPS repository rather than as a standalone generic GPS utility.

## Broadcast control

MockGPS exposes an **external broadcast control interface**. This is the key integration point between the two repositories.

The receiver is:

```text
dev.burixon.mockgps.MockGpsReceiver
```

Supported command values include:

```text
on
set
off
drift
```

The receiver also accepts state-changing extras for boot restoration and notification behavior.

The Android application has a **Broadcast commands** switch. When that option is disabled, MockGPS ignores external broadcast commands.

Because the Termux script relies on this interface, the switch must remain enabled when using `termux-mockgps` for remote control.

## Stored coordinates

The Termux script does **not** maintain a coordinate database or local coordinate file.

The Android MockGPS application stores the selected latitude and longitude itself.

This means:

```sh
termux-mockgps start
```

can reuse the last location selected from the Android application or previously sent through `termux-mockgps set`.

## Notifications and toasts

MockGPS owns the foreground-service notification.

The Termux command can optionally suppress command-related toast messages with:

```sh
-notoast
```

For example:

```sh
termux-mockgps start -notoast
```

The script passes the corresponding setting to the Android application.

## Troubleshooting

### `MockGPS app is not installed.`

Install the companion Android application:

https://github.com/BuriXon-code/MockGPS

### `Android 'am' command not found.`

The script could not find Android's `am` utility. This usually means the command is not being run inside a normal Android/Termux environment.

### Commands are accepted but nothing changes

Check that:

```text
1. MockGPS is installed.
2. MockGPS is selected as the mock-location application.
3. Location permission is granted.
4. Broadcast commands are enabled inside MockGPS.
```

### `set` changes nothing while mocking

Make sure the MockGPS foreground service is still running and that the application is configured as the mock-location provider.

### MockGPS does not restore after reboot

Check the application's **Start on boot** setting, or enable it with:

```sh
termux-mockgps boot on
```

Android OEM battery and background restrictions may also affect automatic restoration.

## Companion application

Android application:

**[BuriXon-code/MockGPS](https://github.com/BuriXon-code/MockGPS)**

This repository provides the Android-side implementation, including:

- map-based location selection,
- mock providers,
- foreground service,
- persistent state,
- reboot restoration,
- external broadcast control.

## Links

- **MockGPS:** https://github.com/BuriXon-code/MockGPS
- **termux-mockgps:** https://github.com/BuriXon-code/termux-mockgps
- **Website:** https://burixon.dev/MockGPS/
- **Bug reports:** https://burixon.dev/bugreport/#termux-mockgps
- **Contact:** https://burixon.dev/contact/
- **Donations:** https://buycoffee.to/burixon-code

## License

GPL-3.0.

See [`LICENSE`](LICENSE) for the full license text.

## Support
### Contact me:
For any issues, suggestions, or questions, reach out via:

- *Email:* support@burixon.dev
- *Contact form:* [Click here](https://burixon.dev/contact/)
- *Bug reports:* [Click here](https://burixon.dev/bugreport/#termux-mockgps)

### Support me:
If you find this script useful, consider supporting my work by making a donation:

[**Donations**](https://burixon.dev/donate/)

Your contributions help in developing new projects and improving existing tools!
