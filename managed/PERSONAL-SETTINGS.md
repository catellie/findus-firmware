# Findus personal settings

Findus has one household layer and one profile per child or owner. Standalone
households edit these in Google Sheets. Managed households keep equivalent
per-child settings in the Slack Canvas; installation-wide settings such as
Wi-Fi, timezone, default volume, and service credentials remain managed by the
installation.

The profile belongs to the person, not to a particular box: a child's Findus is
a Findus instance belonging to that child. With equivalent settings, instances
must use the same persona and response rules across hardware. Screen
shape/resolution and available sensors affect presentation and capabilities,
not personality. Individual generated replies can vary; they need not be
word-for-word identical.

Do not store passwords, API keys, medical records, or other secrets in personal
context. These settings are for ordinary preferences and conversational facts.

## The three language settings

| Setting | Meaning | Example |
| --- | --- | --- |
| `conversation_language` / `tutor_language` | Language Findus must use for replies | `en-GB` |
| `conversation_languages` | Languages Findus expects to hear, primary first | `en-GB,sv-SE` |
| `interface_locale` | Stored local phrases and screen messages | `en-GB` |

A Swedish-speaking child practicing English can use `en-GB` for replies and
the interface while retaining `en-GB,sv-SE` for input. Findus then understands
Swedish but persistently answers in English.

Reviewed language packs currently exist for `sv-SE`, `en-GB`, and `cmn-CN`.
Other locale tags may work in conversation, but local phrase and font coverage
must be qualified separately.

## Standalone Google Sheet

Open **Findus > Personal settings guide** for the generated reference and
examples. Choose **Findus > Check personal settings** after editing. The checker
reports unknown names, likely spelling mistakes, duplicates, malformed locales
and dates, invalid ranges, and unsupported options. Enum and numeric cells also
receive suggestions, while cell notes explain the setting and show an example.

### Household settings

| Setting | Default/example | Use |
| --- | --- | --- |
| `timezone` | `Europe/Stockholm` | Household IANA timezone |
| `default_locale` | `sv-SE` | Fallback locale |
| `interface_locale` | blank | Screen and local phrases; inherits `default_locale` |
| `conversation_language` | `sv-SE` | Household reply language |
| `conversation_languages` | `sv-SE,en-GB` | Expected input languages |
| `shared_prompt` | `Keep answers playful and brief.` | Extra household-wide behavior |
| `shared_interests` | `animals; space; building things` | Topics shared by the family |

### Additional Wi-Fi networks

Additional Wi-Fi networks live in columns D–E of **Household**, beside the
key/value settings. Enter one SSID and password per row; leave the password
blank for an open network. Passwords are visible to anyone who can read the
Sheet. The cells use plain text to preserve leading zeroes and spaces.

After a successful online startup, Findus saves these networks locally for
future connections. The first connection still comes from the device setup
page. Locally saved networks and their priority are retained; duplicate SSIDs,
invalid credentials, or additions exceeding the device's ten-network capacity
are rejected. Removing a row does not erase a network already saved on a device;
use the local setup page to remove it. This feature needs the updated bridge
deployment and firmware; older firmware ignores the additional table.

### Profile settings

| Setting | Default/example | Use |
| --- | --- | --- |
| `profile_id` | generated | Stable identifier; do not edit |
| `profile_name` | `Alex` | Spoken and displayed name |
| `date_of_birth` | `2019-04-26` | Age calculation; use `YYYY-MM-DD` |
| `interests` | `trains; cats; Minecraft` | Personal conversation topics |
| `personal_context` | `Has a dog named Buster.` | Short useful facts |
| `timezone` | blank | Personal timezone; inherits household |
| `conversation_language` | blank | Reply language; inherits household |
| `conversation_languages` | blank | Expected input languages; inherits household |
| `interface_locale` | blank | Screen/local phrase locale; inherits household |
| `voice` | `Leda` | Gemini Live voice name |
| `volume` | `70` | Default volume, `0`–`100` |
| `interaction_level` | `age_based` | `age_based`, `younger`, or `older` |
| `memory_level` | `normal` | `short`, `normal`, or `long` |
| `character_pack` | blank | Optional character pack; blank means Findus |
| `favourite_music` | `Clocks by Coldplay; Ghostbusters by Ray Parker Jr.` | Up to 12 tracks/artists |
| `story_playlists` | Spotify playlist URL | Up to 6 story playlists |

Separate list entries with semicolons or new lines. Profile values override
household values; leaving an inheritable value blank keeps it aligned with
future household changes.

With `interaction_level=age_based`, firmware calculates age from a valid supplied
birth date and the current local date, then selects guidance for vocabulary,
explanation depth, examples and humour: simple and concrete for young children,
more explanatory for older children, and non-patronising for teenagers and adults.
Findus remains the same playful cat. A blank birth date is valid and uses an
accessible age-neutral style; Findus should not invent an age. `younger` and
`older` explicitly request a simpler or more mature presentation without changing
the person's actual age or the safety boundaries. These are model instructions,
not a deterministic reading-level guarantee. Invalid/future dates or an
unsynchronised device clock leave the configured age unknown rather than asking
the model to invent one.

Age and personality are separate choices. Managed Canvas supports
`personality: findus` or `personality: assistant`; the standalone Sheet selects
the same choice with `character_pack` (`assistant`, or blank/`findus` for Findus).
Findus remains a mischievous cat at every age, with increasingly mature
explanations. Assistant omits that character but follows the same age/style
guidance. There is no automatic birthday cutoff that changes personality.
This choice does not change managed versus standalone operation or the selected
voice. Prefer an explicit parent choice if an older child no longer wants the
cat character.

## Managed Slack Canvas

The Canvas remains readable prose. Put inherited structured values in
`Profile Defaults`, before `Per-Child Notes`, and keep only differences inside
each child section:

```markdown
# Profile Defaults
- personality: findus
- voice: Leda
- memory_level: normal
- conversation_languages: sv-SE,en-GB
- story_playlists: https://open.spotify.com/playlist/...

# Per-Child Notes

## Alex
- tutor_language: en-GB
- interface_locale: en-GB
- favourite_music: Clocks by Coldplay; Ghostbusters by Ray Parker Jr.

Alex likes trains and animals.
They have cousins in both Sweden and the UK.
```

Supported structured keys are `personality`, `prompt`, `max_tokens` (32–2048;
normally omit it), `voice`, `memory_level`, `tutor_language`,
`conversation_languages`, `interface_locale`, `favourite_music`, and
`story_playlists`, plus `reply_language_policy` in firmware supporting that
option. `conversation_language` and `favorite_music` are accepted
aliases.

`reply_language_policy` defaults to `fixed`: every reply uses `tutor_language`
regardless of the input language. Opt a profile into `follow_speaker` to reply
in the language the person clearly uses, honour explicit translation/practice
or language-switch requests, and retain that target until another clear cue.
Names, quotes, song titles and ambiguous short answers are not switch cues.
This policy overrides conflicting language roleplay in the shared prompt;
it does not change personality, safety, voice, screen language or local phrases.
The selection is model-guided using conversation context, not a deterministic
language detector or a persisted language choice across reboot. Keep the
recognition hints in `conversation_languages` and use `tutor_language` as the
fallback when the first input is unclear. The Canvas option needs the matching
firmware update; older builds do not recognise it. The standalone Sheet does
not yet expose this option.

Current Canvas formatting precaution (FND-081): enter these two settings as
separate plain paragraphs, not a bullet list immediately followed by prose.
Slack's self-closing break/list HTML can join the last value to personal notes,
causing a validation warning and cached-config fallback. Plain paragraphs were
verified through the real HTML parser and on Lena's managed device.

Defaults may contain `personality`, `max_tokens`, `voice`, `memory_level`, the
three language fields, `reply_language_policy`, `favourite_music`, and
`story_playlists`. A non-empty
child value replaces the corresponding default. Keep shared behavioral prose
in `System Prompt`; keep names, interests, personal facts, and genuinely
personal media preferences in the child section.

Other prose becomes personal context. A colon-bearing key that closely
resembles a supported setting produces a warning with a spelling suggestion
when the Canvas is fetched. Invalid enums, token ranges, and locale forms are
also rejected in favor of safe defaults. Findus reports the collected problems
to the bound Slack channel. This preserves ordinary prose while catching
likely mistakes.

Slack Canvas cannot execute our validator inside the document. Slack can host
workflows and an app can update a Canvas through the Web API, so a future Block
Kit profile editor could offer dropdowns and submission-time errors. For now,
the Canvas template plus fetch-time validation is smaller and transparent.

## Applying changes

Most remote settings apply at the next configuration refresh or boot. Changing
`interface_locale` may generate and install a new local phrase/font pack.
Credentials, Wi-Fi, and enrollment use their separate setup flows.
