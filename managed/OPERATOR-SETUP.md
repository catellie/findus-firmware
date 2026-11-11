# Findus Managed: installation operator guide

Give parents [START-HERE.md](START-HERE.md). This guide is for the person who
maintains Slack, provider credentials, device assignments and installation keys.
It describes the current command-line workflow, not a finished web wizard.

Generic managed firmware contains no installation secrets. It differs from
standalone/Household and from old private-managed firmware with embedded
installation configuration. Do not use a private OTA image as a generic
factory installer.

## End-to-end operator checklist

For a new installation, the complete path is:

1. Choose the permanent installation ID and recovery owner.
2. Create the private service payload and validate Slack, Canvas and provider
   access independently.
3. Create one installation signing key and verify its encrypted backup.
4. Verify the release checksums and flash the exact generic-managed factory
   image for the board.
5. Collect Device, Board, public key and the six-digit on-device code.
6. Add the exact device/profile/channel mappings to the private payload.
7. Seal a revision-1 package and give the parent only the complete enrollment
   wrapper.
8. Confirm greeting, profile, current Slack status, conversation and restart.
9. Record the device's accepted revision, board, installation ID and recovery
   material location without copying secrets into Slack.

For another device in the same installation, reuse the installation ID and
signing key, but collect that device's own key/code and track its revision
independently.

## 1. Choose an installation and recovery owner

Use one stable installation ID, such as `family.example`, for the installation's
devices. It is an identifier, not a DNS ownership check or service endpoint.
Keep the same ID and signing key when adding devices or changing assignment.

Decide who retains the signing key, encrypted backup, backup passphrase,
private payload and per-device revision record. A packaged recovery/export
application is not available yet; do not promise lost-key recovery without
those materials.

The tools below need this source checkout, Node.js with built-in crypto, and
OpenSSL. USB automation also needs Python; direct serial access needs pyserial,
whereas a running Findus monitor supplies its own USB connection. Parents
using the hotspot do not need these tools.

## 2. Prepare the private service configuration

For an existing private-managed installation, export configuration without
printing credentials:

```sh
scripts/export_findus_managed_payload.sh /absolute/private/path/configuration.json
```

The exporter needs the existing private header and creates a mode-0600 file.
It is a migration convenience, not a prerequisite for a new installation.

For a new installation, copy `configuration.example.json` to a private directory
outside the repository and fill in your values. It is a schema-valid blank
template, **not a working service configuration**. Restrict private files to
their owner. Never paste plaintext payloads into Slack, the portal, public
issues or committed source files.

| Configuration | Supply and verify |
| --- | --- |
| `conversation_api_key` | Installation's Gemini credential for managed Live/conversation. |
| `speech_api_key` | Google speech fallback credential; the runtime calls Speech-to-Text and Text-to-Speech. Verify both if fallback is expected. |
| `slack_token`, `default_channel` | App credential and assigned channel ID for parent-facing status/messages. |
| `slack_canvas_id`, `enable_slack_canvas` | Canvas file ID and `true` for remote settings; grant the app access to that Canvas. |
| `identity.device_children` | Exact MAC to child name; an omitted mask defaults to all `ff`, matching only that MAC. |
| `identity.child_channels` | Exactly matching child name to Slack channel ID. |
| `identity.device_boards` | Exact MAC to runtime board ID copied from setup, not guessed from the enclosure. |
| `contacts` | Parent aliases/role and actual Slack user ID when parent polling/voice addressing is enabled. |
| `posix_timezone` | Explicit installation timezone rule; `UTC0` in the example is not a local-time guess. |
| `wifi` | Optional private list of `{ "ssid": "…", "password": "…" }`; otherwise bootstrap in the portal. |
| Spotify client ID/secret | Optional pair for catalog requests; listener account pairing is a separate Connect step. |
| Archive/telemetry URL, developer token | Optional installation services; leave unused values empty rather than copying another family's endpoints. |
| `ota_releases_url` | Generic-managed channel only; the sealing command explicitly sets it. |

Google credentials are not interchangeable merely because both are keys. Test
the exact APIs required. Empty credential sets disable capabilities; a valid
schema alone does not prove conversation or Slack access.

Arrange Slack app installation, channel membership and Canvas sharing first.
The implementation uses `files.info` and authorized Canvas download; parent
status uses chat, pins, channel history and thread APIs. Validate access for
your channel type rather than assuming that one successful post proves all
permissions. Existing installations can reuse their proven app setup. New
Slack/provider accounts remain an operator task, not something enrollment
automates.

### New-installation service checklist

1. Create an app in the intended Slack workspace using its approved app
   management process. Request the scopes needed for the methods Findus uses:
   `files:read` for Canvas download, `chat:write` for status/messages, and
   `pins:read`/`pins:write` for pinned status. Add the relevant history scope
   for your channels (`channels:history` for public, `groups:history` for
   private; DM types use their own history scopes). See Slack's
   [OAuth installation guide](https://docs.slack.dev/authentication/installing-with-oauth/),
   [file access](https://docs.slack.dev/reference/methods/files.info/),
   [pin reads](https://docs.slack.dev/reference/methods/pins.list/),
   [pin writes](https://docs.slack.dev/reference/methods/pins.add/) and
   [thread access](https://docs.slack.dev/reference/methods/conversations.replies/).
2. Install/reinstall the app after scope changes, add it to the intended
   channels, and share the configuration Canvas with it. Store the resulting
   bot token privately as `slack_token`, not the app's signing secret or OAuth
   client secret. Record channel, Canvas file and parent-user IDs separately.
3. Create the managed Gemini credential using Google's
   [API-key instructions](https://ai.google.dev/gemini-api/docs/api-key), then
   place it in `conversation_api_key` in the private payload. Do not use a key
   from an unrelated work/project account without authorization.
4. If conventional speech fallback is required, configure the project and
   credential for both [Speech-to-Text](https://docs.cloud.google.com/speech-to-text/docs/setup)
   and [Text-to-Speech](https://docs.cloud.google.com/text-to-speech/docs/get-started).
   Follow their API-enablement, authentication and billing prerequisites; a
   Gemini-only key is not proof that these APIs work.
5. Start optional Spotify, archive and telemetry integrations only when their
   own credentials/endpoints are ready. Seven is installation-specific: reuse
   the operator's configured service rather than inventing a public endpoint.
6. Validate a canary's current Canvas load, Slack status/pin/thread access,
   speech response and optional integrations before handing the installation
   to a parent. Managed setup has no equivalent of the Sheet's Test API button.

## 3. Prepare the profile Canvas

Use a shared `System Prompt`, inherited `Profile Defaults`, then child sections
under `Per-Child Notes`:

```markdown
# System Prompt
You are Findus, a playful, kind companion. Keep answers child-friendly.

# Profile Defaults
- personality: findus
- voice: Leda
- memory_level: normal
- tutor_language: sv-SE
- conversation_languages: sv-SE,en-GB
- interface_locale: sv-SE

# Per-Child Notes

## Example child
Likes animals and building things.
```

Replace the child name consistently in the payload and Canvas. Use `cmn-CN`
for Mandarin where appropriate; adding it to `conversation_languages` alone
does not change preferred reply/screen language. Keep secrets out of prose.

The full field reference is `docs/findus-personal-settings.md` in source, or
`PERSONAL-SETTINGS.md` in the release archive. Invalid structured settings
produce diagnostics; ordinary prose is not a substitute for a supported key.

## 4. Create and back up the signing key

Do this **once per new installation**, in a new private directory. Do not
regenerate a key to update an enrolled device. With that directory as the shell's
working directory:

```sh
umask 077
if [ -e installation-signing-key.pem ]; then
  echo 'Existing key retained; do not overwrite it.'
else
  openssl ecparam -name prime256v1 -genkey -noout \
    -out installation-signing-key.pem
fi
```

Create an encrypted backup using a new output name. OpenSSL prompts for its
passphrase; do not put it on the command line:

```sh
openssl ec -aes256 -in installation-signing-key.pem \
  -out installation-signing-key.backup.pem
```

Keep the passphrase separately and verify recovery before relying on it. The
current packager reads an unencrypted working PEM; protect it with permissions
and local storage security. It has no passphrase prompt for encrypted keys.
Never package or give either key to parents as enrollment material. Do not
replace an existing backup blindly.

## 5. Identify the device and seal its package

Collect Device, Board and the complete public key from setup, plus the six-digit
code visible on the physical device. Confirm these values with the parent.
The package's runtime board ID is, for example, `esp32-s3-touch-amoled-2.16`,
not the adaptive build selector `waveshare/esp32-s3-findus-adaptive`. A device
public key is 130 hexadecimal characters starting with `04`; it is not a
signing key.

Track revisions separately per device. Start at 1 if no managed bundle exists;
every replacement must exceed the retained revision floor. Reboot reuses the
active revision, but reinstalling it is rejected. New devices can join the same
installation ID without replacing the installation's existing key.

From the source checkout, replace all example arguments:

```sh
node scripts/package_findus_managed_bundle_v2.mjs \
  --payload /absolute/private/path/configuration.json \
  --private-key /absolute/private/path/installation-signing-key.pem \
  --device-public-key 04_REPLACE_WITH_THE_COMPLETE_DEVICE_PUBLIC_KEY \
  --installation-id family.example \
  --device-id 00:11:22:33:44:55 \
  --board-id sp-esp32-s3-1.28-box \
  --revision 1 \
  --ota-releases-url https://catellie.github.io/findus-firmware/ota/generic-managed/stable.json \
  --output /absolute/private/path/device-bundle.json \
  --confirmation-code 000000 \
  --enrollment-output /absolute/private/path/device-enrollment.json
```

The public-key placeholder intentionally cannot be used as a real key. Replace
the MAC/board inside the payload too. Use new output names for revisions you
want to retain. The tool checks its cryptographic round trip and writes mode
0600 files; it does not test live provider access.

Give the parent **the contents of `device-enrollment.json`**. This wrapper has
`version: 2`, `mode: "managed"`, installation ID, signing public key,
confirmation code and encrypted envelope. The inner `device-bundle.json` alone
is not what the portal expects. Payload schema 1 still exists *inside*
encrypted envelope schema 2; never use an old plaintext development envelope
with real credentials.

## 6. Install and verify

### Verify the release directory

Before choosing an image, inspect `release.json` and verify every packaged
binary from the extracted release root:

```sh
shasum -a 256 -c SHA256SUMS
```

Require `source_mode` to be `committed`, confirm the expected version and
source commit, and confirm `configuration` is `generic-managed`. The board's
`build-manifest.txt` must name the same configuration. Reject a working-tree
artifact, a standalone image, a checksum mismatch or a board chosen only by
screen size.

### Check assets as well as the application

An application-only flash preserves the existing asset archive. On an original
Xiaozhi device this can leave generic emoji faces even though Findus firmware
and enrollment work. Before conversion, check both the exact-board application
and its matching Findus graphics/model archive. Existing Findus devices may
already have suitable assets; do not rewrite them without checking.

Back up the device first and verify its partition table. If an asset update is
needed, install only the matching immutable archive, not the whole assets
partition or a factory image over an enrolled device. The partition tail also
stores managed bundles, cached Canvas settings and generated phrases. Check
the erase-aligned archive end against the storage layout and verify the
protected tail is unchanged afterward. Do not interrupt phrase generation to
perform the update. Reboot and verify the intended Findus artwork, assignment
and phrase-pack reuse.

The 2026-10-21 MUMA canary demonstrates this procedure; its offsets are evidence
for that checked 16 MB layout, not universal offsets for other hardware.

Use `START-HERE.md` for the parent hotspot flow. If the correct Wi-Fi is already
saved, leave the new SSID/password fields blank.

For operator-controlled USB setup, first put a new device visibly into setup
mode. With its monitor running:

```sh
python3 scripts/findus_provision.py --device-id 00:11:22:33:44:55 info
python3 scripts/findus_provision.py --device-id 00:11:22:33:44:55 \
  managed install /absolute/private/path/device-enrollment.json
python3 scripts/findus_provision.py --device-id 00:11:22:33:44:55 reboot
```

Without a monitor, supply `--port` and a Python environment with pyserial.
Wi-Fi mutation and first enrollment require visible setup. Enrolled devices
accept higher-revision packages signed by the pinned key while locked. Current
firmware permits USB reboot in either personality; older standalone images may
return `LOCKED` until updated.

Verify the intended greeting, language/time and current Slack configuration
result for that MAC. On a logged canary confirm managed configuration loaded,
expected capabilities enabled and the bundle marked healthy. That marker means
startup accepted the bundle, not that every optional service is reachable.

Run a short spoken exchange, check status/archive delivery, then restart and
check persisted assignment. Test optional music and parent messaging separately.
An old Slack pin, fallback defaults or Wi-Fi success alone is not acceptance.

## 7. Updates and recovery

- Ordinary profile changes belong in the Canvas. New secrets, assignment or
  installation policy require a higher-revision sealed package, not a private
  firmware rebuild.
- Keep the same installation ID and signing key. Do not edit signed JSON or
  generate another key to work around a trust mismatch.
- Application OTA and Wi-Fi reset retain identity. A full erase loses it and
  requires explicit recovery and fresh device-bound enrollment. Never erase
  just because a network/provider is unavailable.
- Wi-Fi failure after submission does not prove package installation failed.
  Inspect state/revision before issuing another package.
- Keep private-managed, generic-managed and standalone OTA channels separate.
  Changing an old private device's OTA URL alone does not migrate it: arrange
  verified bundle provisioning or an explicit manual reinstall.
- Release with `build_findus_generic_managed_release.sh` and follow
  `docs/findus-release.md`. Freeze dependency versions, check the resolved graph,
  verify the update channel and qualify exact boards before publishing.
- Publish a new versioned OTA directory and validate it remotely before moving
  `stable.json`. Canary the exact immutable binary first; changing the stable
  pointer is the fleet rollout, not the staging step.

For protocol/security details see `docs/findus-managed-bundle-contract.md` and
`docs/findus-generic-managed-plan.md`. Current device key storage is not a
promise of protection against invasive physical access.
