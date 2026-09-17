# Findus Household Starter Kit

This kit installs a new, household-owned Findus. It contains generic firmware
and no Wi-Fi passwords, Google keys, Spotify credentials, Slack configuration,
or information about another family.

## Before connecting Findus

1. Open [**Copy the Findus Household Sheet**](https://docs.google.com/spreadsheets/d/1lVdV5u5hUxVgkZZ4KlsHdFo0BH_kcI4B0WyUQ7ghX-8/copy).
2. Make a private copy in the Google account that will own the household.
3. In the copied Sheet, open **Findus > Set up Findus**.
4. Follow stages 1–3 to initialize the Sheet, store and test the household's
   Gemini API key, and deploy its bridge.
5. In stage 4, enter the child or owner's name and create the enrollment bundle.

The Gemini key is stored in private Apps Script properties, not in a Sheet
cell or in this firmware. Spotify is optional and can be configured later.

## Install the correct firmware

Choose by the device's physical appearance. Never try an image merely because
its screen has a similar size.

### Round 1.28-inch Findus

- Board ID: `sp-esp32-s3-1.28-box`
- Initial image: `firmware/round/findus-round-factory.bin`
- Manual OTA/recovery image: `firmware/round/findus-round-ota.bin`

### White Waveshare 2.16-inch AMOLED Findus

- Board ID: `waveshare/esp32-s3-touch-amoled-2.16`
- Initial image:
  `firmware/waveshare-amoled-2.16/findus-waveshare-amoled-2.16-factory.bin`
- Manual OTA/recovery image:
  `firmware/waveshare-amoled-2.16/findus-waveshare-amoled-2.16-ota.bin`

For a new or completely erased device, use the **factory** image at offset
`0x0`. The smaller **OTA** image is not a complete first installation.

The GitHub release page should normally provide a browser installer so a
parent does not need to choose offsets manually. The binaries in this archive
are the recovery and offline path.

The browser installer requires a desktop Chromium-based browser and an HTTPS
page. Disconnect unrelated ESP32 devices before pressing Install, then select
the USB device that was just connected.

## Enroll Findus

1. Restart the newly installed device.
2. Join the Wi-Fi network named `Findus-XXXX` from a phone or computer.
3. Select the household Wi-Fi network and enter its password.
4. Paste the complete enrollment bundle copied from the Sheet.
5. Submit the form and reconnect the phone or computer to its normal Wi-Fi.

Findus registers itself, downloads its profile and generated local language
pack, and then enters Standby. Later devices use the same household Sheet but
receive a newly generated one-time enrollment bundle.

## Verify the download

`starter-kit.json` records the exact source commit, firmware version, supported
boards, and SHA-256 digest of every image. `SHA256SUMS` contains the same image
digests in a form suitable for common checksum tools.

If Findus displays an upgrade warning, open the Sheet setup sidebar before
installing unrelated firmware. Firmware and bridge compatibility are checked
explicitly; do not work around that warning by copying another household's
credentials.
