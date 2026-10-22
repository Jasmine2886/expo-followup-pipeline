# Gotcha quick-reference — browser / login / captcha / coordinates / tooling

## Browser and login

| Symptom | Cause | Fix |
| --- | --- | --- |
| User is logged into their own Chrome, automation browser still shows logged out | Automation browser uses an **isolated profile**; no cookie sharing | Complete login inside the automation session, or fall back to pasted content |
| Third-party one-tap SSO (Google / Apple / Microsoft / social) lands on someone else's account | That identity is bound to a different account in the system | Use phone-code or username/password instead; verify account ownership first |
| Image captcha blocks even in headless mode | Risk control doesn't distinguish headless | Screenshot → interpret → fill → submit. Wrong answers rotate the image; cap at 3 rounds |
| Sandbox `screencapture` reports "could not create image from display" | No screen-recording permission | Can't confirm the window is visible → **ask the user**, don't guess |
| Window opened but URL stays `about:blank` | Navigation lost after session restart | Navigate once more, then read the URL |
| Keychain read denied (e.g. reading encrypted browser cookies) | OS permission | Abandon that route; official login (email/phone code) is faster |

## Coordinates and clicking

- Confirm the coordinate space first: check `window.devicePixelRatio` and screenshot dimensions
  - dpr=1 and screenshot size == viewport size → `getBoundingClientRect()` maps **1:1** to screenshot pixels
- For "click the smallest blue cone"-style captchas: `eval` the captcha image's `getBoundingClientRect()` first, then convert by relative position inside the image
- **Never click based on how a scaled-down preview looks.** Images get shrunk when displayed in chat — clicking where it visually appears will always be off
- `sips --cropOffset` (macOS) semantics are unintuitive; crop position needs one trial run to confirm. When unsure, screenshot the full image. On Linux/Windows use your OS's equivalent (ImageMagick `convert`/`magick`, PowerShell, etc.)

## Time and timezone

- Event app / web timestamps usually render in the **browser's local timezone**, not the event's local timezone
- Convert everything to `TZ_BASE` and state the relationship in the output (e.g. "local = base −4h")
- When a time is ambiguous ("10:30 a.m." — which zone?), derive it from the UTC timestamp in the email attachment or invite. Don't guess

## Task & calendar CLI (when `TODOOL` is a CLI — known traps)

| Trap | Detail |
| --- | --- |
| Passing notification/reminder params to `calendar-create` | Usually unsupported; the whole call fails silently. Those params belong to **update** |
| Not verifying after batch event creation | Failures can return something that looks like success → always read the agenda back |
| Deleting an event without the confirm flag | Returns ok, deletes nothing |
| Updating a time with only one endpoint given | Validation error — start and end must both be supplied |
| Whole-block document replacement | Block IDs change; reusing an old ID always fails → use targeted string replace for text edits |
| Locating a table by keyword scope | Unreliable (may hit a different table) → use a known block ID or filter by content |
| Judging success from `\| head`-truncated output | Version banners get mistaken for success → write to disk, then parse |

## General

- Always **write command output to disk, then parse it.** Never decide from a truncated pipe fragment
- More than 3 failed login rounds → switch to the fallback path (user pastes content / human steps in). Don't grind
- When writing documents, HTML entities like `&ldquo;` get turned into literal text → use typographic quote characters directly
