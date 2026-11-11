# Findus

Findus is a playful voice companion for children, running on a small ESP32-S3
touch device. A child taps Findus and talks naturally: ask curious questions,
continue an interactive story, practise ideas aloud, request music, or share
something that happened. Findus adapts its language and level to the child
while parents retain control over settings and can review what happened.

There are two editions with the same child-facing Findus personality:

| Choose | When it fits | Adult control |
| --- | --- | --- |
| **Household / standalone** | One family wants to own and try Findus directly | A private Google Sheet owned by the household |
| **Generic managed** | A trusted technical person supports several devices or integrations | Slack, a private configuration Canvas, and encrypted device enrollment |

Both firmware variants are reusable and contain no family's credentials. They
use separate setup packages and OTA channels so one edition cannot silently
turn into the other.

## Set up a household Findus

The household edition keeps the family's settings in a private Google Sheet.
The public firmware contains no Wi-Fi passwords, Google keys, Spotify login, or
details about another family.

**Start here: [Set up Findus Household](https://catellie.github.io/findus-firmware/household/)**

You will need:

- a supported Findus device;
- a USB data cable and a desktop computer running Chrome or Edge;
- a Google account for the private household Sheet; and
- the courage to replace the original firmware on a small device from China.

The setup guide first creates the household Sheet and bridge, then identifies
and flashes the exact hardware model, and finally connects the device using a
short-lived enrollment bundle. Never select firmware merely because two boxes
look similar.

### Supported household hardware

- **Round 1.28-inch Findus** — `sp-esp32-s3-1.28-box`
- **MUMA 1.54-inch Findus** — `sp-esp32-s3-1.54-muma`
- **Waveshare 1.54-inch LCD** — `waveshare/esp32-s3-touch-lcd-1.54`
- **Waveshare 1.83-inch portrait LCD** — adaptive Waveshare firmware
- **Waveshare 2.16-inch AMOLED** — adaptive Waveshare firmware

The installer presents a separate button for each build target. Choose by the
exact product and board ID, never merely by screen size.

## Managed family installation

The managed edition uses Slack, a private configuration canvas, and optional
household infrastructure. The reusable generic-managed firmware contains no
installation credentials; its private configuration is installed separately
as an encrypted managed bundle. Managed devices receive updates from their
separate public [stable channel](https://catellie.github.io/findus-firmware/ota/generic-managed/stable.json).
These are not the correct first-install images for a household-owned standalone
Findus.

- [Managed parent setup guide](managed/START-HERE.md)
- [Managed installation-operator guide](managed/OPERATOR-SETUP.md)
- [Download the current release](https://github.com/catellie/findus-firmware/releases/latest)

## Recovery and verification

Each versioned household installer includes full factory images, OTA/recovery
images, build metadata, and SHA-256 checksums. The browser installer chooses the
factory image and flash offset automatically.

Enrolled household devices receive standalone updates from the public
[stable channel](https://catellie.github.io/findus-firmware/ota/standalone/stable.json).
Its versioned manifests and binaries are separate from generic-managed
releases so a device cannot cross between the two configurations by OTA.

Findus is based on the
[xiaozhi-esp32](https://github.com/78/xiaozhi-esp32) framework.
