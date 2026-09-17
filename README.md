# Findus

Findus is a small ESP32-S3 companion for children: press the screen, talk, and
Findus answers with a child-friendly voice. It can hold a conversation, tell
interactive stories, and play requested music and stories through Spotify.

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
- **White Waveshare 2.16-inch AMOLED** — `waveshare/esp32-s3-touch-amoled-2.16`

The installer presents a separate button for each board. A future revision will
add before-and-after photographs showing the factory device and its Findus
screen, plus distinctive ports and labels, before additional similar white
boxes are offered.

## Managed family installation

The managed edition uses Slack, a private configuration canvas, and optional
household infrastructure. Its OTA binaries remain available under
[Releases](https://github.com/catellie/findus-firmware/releases), but they are
not the correct first-install images for a household-owned standalone Findus.

## Recovery and verification

Each versioned household installer includes full factory images, OTA/recovery
images, build metadata, and SHA-256 checksums. The browser installer chooses the
factory image and flash offset automatically.

Findus is based on the
[xiaozhi-esp32](https://github.com/78/xiaozhi-esp32) framework.
