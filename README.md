# termux-mockgps

A minimal POSIX shell command-line interface for controlling the Android [MockGPS](https://github.com/BuriXon-code/MockGPS) application from Termux.

The command is intentionally simple. It does not maintain a configuration file and does not require a separate status or database system.

## Usage

```text
usage: termux-mockgps [options]
Mock the device location.

  start            Start mocking the last saved location.
  start [lat lon]  Start mocking at the specified location.
  stop             Stop mocking and restore normal location.
  set [lat lon]    Set the mock location.
```

Coordinates may also be specified explicitly:

```sh
termux-mockgps start -lat:50.06143 -lon:19.93658
```

## Requirements

* Termux.
* Android 15 or newer.
* The `MockGPS` Android application installed.
* MockGPS selected as the mock-location application in Android Developer Options.

The script is POSIX `sh`.

## Installation

Copy the script to Termux's executable directory:

```sh
cp termux-mockgps "$PREFIX/bin/termux-mockgps"
chmod +x "$PREFIX/bin/termux-mockgps"
```

Test the installation:

```sh
termux-mockgps help
```

## Start

Start MockGPS using the last location stored by the Android application:

```sh
termux-mockgps start
```

Start at a specific location:

```sh
termux-mockgps start 50.06143 19.93658
```

Coordinates are always:

```text
latitude longitude
```

The location is stored by MockGPS itself.

The CLI does not create or maintain a local coordinate file.

## Set

Change the mock location:

```sh
termux-mockgps set 50.06143 19.93658
```

The same command can be written using explicit coordinate options:

```sh
termux-mockgps set -lat:50.06143 -lon:19.93658
```

or

```sh
termux-mockgps set 50.06143,19.93658
```

When MockGPS is running, the new location becomes active automatically.

When MockGPS is stopped, the new coordinates are stored for the next `start`.

The `set` option forces the application to send an additional notification informing about coordinate changes. However, to change the coordinates without issuing a notification, use the additional `-silent` option.

## Stop

Stop mocking:

```sh
termux-mockgps stop
```

This stops the Android foreground service and disables automatic restoration.

## Android permissions

After installing MockGPS, configure Android before using the Termux command.

### Location permission

Open:

```text
Settings
→ Apps
→ MockGPS
→ Permissions
→ Location
```

Grant location access.

### Notification permission

Open:

```text
Settings
→ Apps
→ MockGPS
→ Notifications
```

and allow notifications.

MockGPS uses an Android foreground service, which requires a foreground-service notification. Android 13 and newer also provide users with control over notification permission.

### Mock-location application

Open:

```text
Settings
→ Developer options
→ Select mock location app
```

and select:

```text
MockGPS
```

The mock-location application must be selected before Android will accept injected test locations.

## How it works

`termux-mockgps` communicates with MockGPS using Android's `am` command.

The important commands are:

```text
start → start the Android foreground service
set   → send new coordinates to the Android application
stop  → disable persistent operation and stop the service
```

The Android application remains responsible for storing the last coordinates and managing the location providers.

## Notification

When MockGPS is running, Android shows a small ongoing notification containing the current coordinates:

```text
MockGPS
Location: 50.06143, 19.93658
```

The notification is intentionally low importance and quiet.

Modern Android versions control exactly how ongoing foreground-service notifications can be dismissed and displayed. The application cannot completely override all System UI behavior.

## Reboot

When MockGPS is enabled, the Android application remembers its last location and attempts to restore the service after reboot.

This behavior belongs to MockGPS, not to the Termux script.

Some Android devices apply additional OEM-specific battery or background restrictions, which can affect automatic service restoration.

## Updating MockGPS

MockGPS is currently under active development.

At this stage, updating the Android application may occasionally leave an old service/process state behind.

After installing a new development version, you may need to completely reinstall the Android application and reboot the device:

```text
1. Stop MockGPS.
2. Uninstall MockGPS.
3. Install the new APK.
4. Select MockGPS again as the mock-location app.
5. Re-grant permissions.
6. Reboot the device.
```

This is a current development-stage limitation and is expected to be improved in future versions.

## Troubleshooting

If `start` suddenly stops working after repeated development builds or service restarts, completely uninstall and reinstall MockGPS and reboot the device.

Also check:

```text
Developer options
→ Select mock location app
→ MockGPS
```

and:

```text
Settings
→ Apps
→ MockGPS
→ Permissions
→ Location
```

## Companion project

Android application:

[MockGPS](https://github.com/BuriXon-code/MockGPS)

Termux command:

[termux-mockgps](https://github.com/BuriXon-code/termux-mockgps)

## License

GPL-3.0.

See the `LICENSE` file for the full license text.

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
