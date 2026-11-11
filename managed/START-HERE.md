# Set up Findus Managed

This guide is for a parent adding a device to an installation maintained by
someone they trust. Managed Findus uses Slack and the installation's configured
services. It does not use the standalone Household Sheet or its claim bundle.

The generic firmware contains no installation credentials. An encrypted
enrollment package assigns your particular device to the installation and
child. The installation operator prepares it; you should not need their
signing key, provider keys, or source code.

Setup is currently operator-assisted. A self-service installation/key-recovery
wizard is not yet available. Operators use [OPERATOR-SETUP.md](OPERATOR-SETUP.md).

## The short version

1. The operator confirms the exact board and supplies its generic-managed
   factory image.
2. Install that image, restart Findus and join its `Findus-XXXX` setup Wi-Fi.
3. Send the displayed Device, Board and complete public key to the operator.
4. Read the six-digit confirmation code from Findus itself, not the web page.
5. The operator returns one complete, device-bound enrollment package.
6. Paste that package in managed setup, add Wi-Fi only if no saved network is
   usable, then press **Connect** once.
7. Wait for Findus to restart, greet the intended child and reach Standby.
8. Ask the operator to confirm the current Slack configuration result, then
   test one conversation and one restart.

The rest of this guide explains how to do those steps safely and recover when
one of them does not complete.

## 1. Before you start

Have these ready:

- the Findus device, a USB data cable and a computer;
- the child/profile name and contact details for the installation operator;
- the operator's exact firmware choice for your hardware;
- local Wi-Fi credentials, unless a working network is already saved or the
  operator has included it in the package.

Choose **generic managed**, not **standalone/Household**. Old private-managed
OTA binaries are not generic first-install images. Do not choose firmware just
because its screen size looks similar.

## 2. Install the correct image

If the operator already installed Findus, skip to step 3. Otherwise obtain the
exact factory image and flashing instructions from them. There is not yet a
managed equivalent of the standalone Sheet wizard.

The release archive uses these names:

| Hardware | Firmware directory | Board value shown in setup |
| --- | --- | --- |
| Round 1.28-inch | `firmware/round/` | `sp-esp32-s3-1.28-box` |
| MUMA 1.54-inch | `firmware/muma-1.54/` | `sp-esp32-s3-1.54-muma` |
| Waveshare 1.54-inch LCD | `firmware/waveshare-lcd-1.54/` | `esp32-s3-touch-lcd-1.54` |
| Adaptive Waveshare 1.83-inch portrait LCD or 2.16-inch AMOLED | `firmware/waveshare-adaptive/` | `esp32-s3-touch-lcd-1.83` or `esp32-s3-touch-amoled-2.16` |

Each directory contains `findus-…-generic-managed-factory.bin` and a smaller
`findus-…-generic-managed-ota.bin`. A complete first installation uses the
**factory** image at offset `0x0`. An ordinary update uses the application-only
image through the supported update procedure. Factory flashing can replace
saved settings: do not use it on an enrolled device without the operator's
approval and recovery plan.

These are build targets, not a promise that every feature has been physically
qualified on every board. Ask which hardware is approved for your release.

### Verify the release before flashing

Use an intact release directory. `release.json` identifies the firmware
version, source commit, managed-bundle compatibility and exact supported
boards. `SHA256SUMS` covers every factory and application-only image. On macOS
or Linux, from the extracted `Findus-Generic-Managed-vVERSION` directory, run:

```sh
shasum -a 256 -c SHA256SUMS
```

Every entry must report `OK`. If it does not, download or extract the release
again; do not flash the mismatching image. Windows users can compare
`Get-FileHash -Algorithm SHA256` with the corresponding line in `SHA256SUMS`.

The factory image is a complete image written at offset `0x0`. The OTA image
is only the application and is suitable for an already compatible Findus. It
cannot turn an erased or stock device into managed Findus. If no approved
browser installer is supplied for this release, the operator should perform
the initial flash with their normal verified ESP32-S3 tool rather than asking
a parent to guess ports, offsets or button sequences.

## 3. Open managed setup

1. Restart Findus and read the setup message.
2. If it explicitly asks you to press **BOOT** to enter **Managed setup**,
   confirm only when ready. That recovery action clears saved Wi-Fi and
   restarts setup; it is not a full factory erase.
3. Join `Findus-XXXX`. This temporary Wi-Fi has no internet access; that is
   expected.
4. If the setup window does not open, visit `http://192.168.4.1/` in a browser.
5. Check that the Findus section says **managed**. If it asks for a bundle
   copied from a Google Sheet, stop and ask the operator to check the firmware.

If Findus already runs normally, do not clear networks just to find a hotspot.
Ask the operator to use the existing connection or guarded USB setup. Button
positions vary between models; use your board's labels, not another box's
button sequence.

## 4. Ask for this device's enrollment package

Copy these public values from setup to the operator:

- **Device**;
- **Board**;
- **Device public key**, including all text if it wraps across lines.

Also read the **six-digit confirmation code from the physical Findus screen**.
It is deliberately not returned by the web page or USB identity command.
Confirm the installation and child's name with the operator before installing.

You may return the computer to normal Wi-Fi while exchanging these values,
then rejoin `Findus-XXXX` to finish. Captive setup windows can close on a network
switch; reopen the browser page after reconnecting.

The operator returns a complete **managed enrollment package**, not an API key,
private signing key, standalone claim bundle, or just the encrypted envelope.
Managed packages are device-bound and revision-controlled; they do not use
the standalone Sheet's 30-minute claim procedure.

## 5. Submit the package and Wi-Fi settings

1. Rejoin `Findus-XXXX` and reopen `http://192.168.4.1/` if necessary.
2. Paste the complete JSON package into **Enrollment bundle**.
3. If the correct network appears under **Saved Wi-Fi**, leave the new
   SSID/password fields blank. Do not re-add it just because those fields
   exist. Otherwise enter a reachable network and its password. The operator
   may also have supplied networks in the package.
4. Press **Connect** once. Wait for explicit success or error before retrying.
5. Reconnect the computer to normal Wi-Fi when Findus restarts or its setup
   network disappears.

Current candidate firmware schedules that restart on Findus itself after
saving successfully; closing the captive window does not cancel it. If you
are only adding a network to an already-enrolled device, leave Enrollment
bundle blank: the installed package is retained, with no new revision needed.
Older firmware can remain in the hotspot if the success window closes early;
one normal restart then uses the saved network.

Do not unplug Findus while it saves a package or generated phrases. A long
package is normal: paste all of it without code fences, added quotation marks
around the whole object, or an incomplete selection.

## 6. Check that setup actually completed

Successful Wi-Fi setup alone does **not** prove managed enrollment succeeded.

1. Findus leaves repair/setup and greets the intended child.
2. It reaches Standby. The first boot may prepare phrases; later boots reuse
   an unchanged installed pack rather than generating it again.
3. The operator confirms a current configuration result for this device in
   its Slack channel. Managed installations do not have the standalone Sheet's
   green Devices row. An old pinned message is not proof of this boot: check
   the device identity and current update.
4. Try a short spoken exchange, then restart once and confirm the assignment
   and Wi-Fi persist. Test optional music/parent messages only if enabled.

Tell the operator if the name, language, voice or local time is wrong. Ordinary
personal settings belong in the managed configuration Canvas; credentials and
device assignment belong in the installation package.

## If something goes wrong

| What you see | What to do |
| --- | --- |
| No hotspot, but Findus runs normally | Saved Wi-Fi may already work. Do not reset it unnecessarily. |
| Page says standalone or mentions a Sheet bundle | Stop and check firmware personality and exact board with the operator. |
| No Enrollment bundle field | Reopen managed setup. If it remains Wi-Fi-only, have the operator check firmware/version and setup state. |
| “This enrollment bundle version is not supported” | Obtain the full version-2 managed enrollment package, not just its inner envelope or a development bundle. |
| Code, trust, device or board mismatch | Recheck the device values with the operator. Do not edit the signed package. |
| An older revision is rejected | The operator must issue a higher revision using the same installation key. |
| “Failed to connect to the Access Point” | Check reachability/password. The package may already have installed before Wi-Fi failed; inspect state before issuing another revision. |
| “Configuration successful” but Findus still asks for repair | Wi-Fi success is not enrollment success. Tell the operator the diagnostic code shown. |
| `device_repair`, missing identity, or repeated reboot | Stop retrying blindly; send the diagnostic code to the operator. Do not erase flash or open the case as a setup workaround. |
| Computer lost internet during setup | Expected on `Findus-XXXX`; reconnect to normal Wi-Fi afterward. |
| Spotify search succeeds but playback cannot start | Pair this physical Findus as a Spotify Connect device in the Spotify app. Pairing belongs to each device, not merely to the household configuration. |
| Amber `Connecting again (2/2)...` | Findus is making its single fresh connection attempt. Wait for recovery or the final network error before pressing again. |
| Findus remains on `Connecting...` after the retry/error | Restart once and tell the operator the exact status and time. Do not repeatedly erase Wi-Fi or reinstall the bundle. |

Never post passwords, provider keys, decrypted payloads or signing keys in a
public issue. The operator should retain recovery material securely.

## Normal changes after installation

- Change name, languages, voice, age context and interests in the managed
  configuration Canvas; they do not require reflashing or a new signing key.
- Changing assignment, credentials or installation policy requires a
  higher-revision package signed by the same installation key.
- Add a new Wi-Fi through managed setup with the Enrollment bundle field left
  blank. A successful save schedules Findus's own restart and retains its
  installed identity.
- Normal firmware updates use the generic-managed OTA channel and retain the
  package, Wi-Fi and generated phrases. Never install a standalone OTA over a
  managed device.
