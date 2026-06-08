# Thoughts Remote Config v1 — Complete Usage & Safety Guide

This guide explains how to use `remote_config.json` for Thoughts Android.

Remote Config is for safe live content and controlled feature flags. It can change content, show cards, show notices, override selected copy, and disable selected non-sensitive features. It cannot add new Flutter code, native Android features, permissions, storage logic, encryption, widgets, or destructive behavior unless that behavior already exists inside the installed APK.

## Core rule

Remote Config can control data and presentation. It must not control private user data, destructive actions, App Lock, screen protection, or database behavior.

Safe:
- Show a card.
- Show a notice.
- Change selected help text.
- Disable a non-sensitive UI action temporarily.
- Link to a trusted page.
- Open prebuilt safe app actions such as Backup, Updates, Guides, or Compose.

Unsafe:
- Delete notes.
- Clear Recently Deleted.
- Disable App Lock.
- Disable protected screen previews.
- Switch writing spaces.
- Import backups automatically.
- Run remote code.
- Render arbitrary HTML.
- Execute JavaScript.
- Add native Android permissions.
- Change encryption logic remotely.

## File location

Recommended GitHub raw URL:

```text
https://raw.githubusercontent.com/dheeraz101/webstore-thoughts/refs/heads/main/remote_config.json
```

Your app should fetch this URL, cache the successful response, and use cached config when offline.

## Minimal valid file

```json
{
  "schema": "thoughts.remote.v1",
  "enabled": true,
  "minAppVersion": "1.5.0",
  "lastUpdated": "2026-06-05"
}
```

This does nothing visible, but it is valid.

## Complete structure

```json
{
  "schema": "thoughts.remote.v1",
  "enabled": true,
  "minAppVersion": "1.5.0",
  "lastUpdated": "2026-06-05",
  "featureFlags": {},
  "copyOverrides": {},
  "cards": [],
  "notices": []
}
```

## Root fields

### `schema`

Required.

```json
"schema": "thoughts.remote.v1"
```

The app ignores the file if this does not match exactly.

### `enabled`

Optional. Defaults to enabled if missing.

```json
"enabled": true
```

Set to false to make the app ignore the config:

```json
"enabled": false
```

### `minAppVersion`

Optional.

```json
"minAppVersion": "1.5.0"
```

If the installed app version is older than this, it ignores the config. Use this when remote cards require code added in a newer APK.

### `lastUpdated`

Informational. Use ISO date.

```json
"lastUpdated": "2026-06-05"
```

## Feature flags

Feature flags are boolean switches for safe, prebuilt behavior.

Example:

```json
"featureFlags": {
  "showRemoteCards": true,
  "showRemoteNotices": true,
  "backupImportEnabled": true,
  "shareImageEnabled": true
}
```

Supported flags in v1:

```text
showRemoteCards
showRemoteNotices
showKnownIssues
showWritingPrompts
showReleaseHighlight
backupImportEnabled
shareImageEnabled
launchKeyboardCardEnabled
remoteTipsEnabled
appLinksCardEnabled
```

### `showRemoteCards`

Controls whether remote cards appear.

```json
"showRemoteCards": true
```

Set false to hide all remote cards:

```json
"showRemoteCards": false
```

### `showRemoteNotices`

Controls whether remote in-app notices can appear.

```json
"showRemoteNotices": true
```

### `backupImportEnabled`

Safe kill switch for Import Backup.

```json
"backupImportEnabled": false
```

When false, the app should show Import Backup as temporarily unavailable.

### `shareImageEnabled`

Safe kill switch for Share as Image.

```json
"shareImageEnabled": false
```

When false, the app should prevent image sharing and show a short notice.

## Copy overrides

Copy overrides let you safely replace specific text strings without publishing a new APK.

Supported copy keys in v1:

```text
backupGuideText
settingsBackupSubtitle
settingsNoticeSubtitle
appLinksHelpText
privacyReminderText
releaseHighlightText
```

Example:

```json
"copyOverrides": {
  "backupGuideText": "Your notes stay on this device. Export a backup before switching phones, clearing app data, or reinstalling."
}
```

Rules:
- Use only approved keys.
- Keep values short.
- Do not include private user data.
- Do not include scary or manipulative copy.
- Do not use raw HTML.
- Do not include tracking links.

## Remote cards

Remote cards are small UI cards shown in safe surfaces such as Settings.

Example:

```json
"cards": [
  {
    "id": "backup-safety-001",
    "enabled": true,
    "surface": "settings",
    "kind": "backup",
    "priority": 100,
    "title": "Protect your notes",
    "body": "Thoughts stores notes locally. Export a backup before switching phones or clearing app data.",
    "actionLabel": "Open Backup",
    "action": "backup",
    "maxShows": 8,
    "cooldownHours": 48
  }
]
```

### Card fields

#### `id`

Required. Must be unique and stable.

```json
"id": "backup-safety-001"
```

Change the ID if you want the card to be treated as a new campaign.

#### `enabled`

Optional.

```json
"enabled": true
```

Set false to disable the card without deleting it.

#### `surface`

Where the card appears.

Supported v1 surfaces:

```text
settings
home
updates
privacy
backup
```

Your current implementation renders `settings`. Other surfaces should be added only when the app UI has a safe place for them.

#### `kind`

Controls icon/style.

Supported kinds:

```text
info
warning
backup
prompt
release
issue
```

#### `priority`

Higher priority appears first.

```json
"priority": 100
```

Use ranges:
- 100: important safety / backup card
- 80: prompt / release highlight
- 40: known issue / informational
- 10: small tip

#### `title`

Short title. Recommended under 70 characters.

#### `body`

Short body text. Recommended under 220 characters.

#### `actionLabel`

Button/action label. Keep under 32 characters.

#### `action`

Safe action to run when tapped.

Supported safe actions:

```text
compose
new
note
backup
settings
updates
update
notifications
notices
about
guides
whats-new
whatsnew
changelog
release-notes
github
home
open
dismiss
```

Trusted HTTPS URLs may be allowed only for these hosts:

```text
writethoughts.netlify.app
github.com
raw.githubusercontent.com
dheeraz.netlify.app
```

Do not use arbitrary URLs.

#### `maxShows`

Maximum times this card can be shown/tapped before it stops appearing.

Recommended:
- Important backup card: 5–8
- Known issue card: 3–4
- Writing prompt: 1–5

#### `cooldownHours`

Minimum hours between appearances.

Recommended:
- Backup reminder: 48–72
- Prompt: 24–72
- Known issue: 96–168

## Remote notices

Remote notices are small in-app notification pills.

Example:

```json
"notices": [
  {
    "id": "backup-reminder-remote-001",
    "enabled": true,
    "priority": 100,
    "message": "Your notes stay local. Export a backup before switching phones.",
    "actionLabel": "Backup",
    "action": "backup",
    "maxShows": 2,
    "cooldownHours": 72
  }
]
```

### Notice fields

#### `id`

Required, unique, stable.

#### `enabled`

Set false to disable.

#### `priority`

Higher appears first.

#### `message`

Short notice text. Recommended under 160 characters.

#### `actionLabel`

Optional label for the action.

#### `action`

Optional safe action. Use empty string for no action:

```json
"action": ""
```

If action is empty, it should still be allowed to show as a message-only notice.

#### `maxShows`

Maximum displays.

Recommended: 1–3.

#### `cooldownHours`

Recommended: 48–168.

## Real-world use cases

### 1. Disable a broken feature temporarily

```json
{
  "schema": "thoughts.remote.v1",
  "enabled": true,
  "minAppVersion": "1.5.0",
  "featureFlags": {
    "backupImportEnabled": false
  }
}
```

Use when Import Backup has a bug and you need to prevent damage until the next APK.

### 2. Show a backup safety card

```json
{
  "schema": "thoughts.remote.v1",
  "enabled": true,
  "minAppVersion": "1.5.0",
  "cards": [
    {
      "id": "backup-safety-001",
      "enabled": true,
      "surface": "settings",
      "kind": "backup",
      "priority": 100,
      "title": "Protect your notes",
      "body": "Export a backup before switching phones, clearing app data, or reinstalling.",
      "actionLabel": "Open Backup",
      "action": "backup",
      "maxShows": 8,
      "cooldownHours": 48
    }
  ]
}
```

### 3. Show a writing prompt

```json
{
  "schema": "thoughts.remote.v1",
  "enabled": true,
  "cards": [
    {
      "id": "prompt-001",
      "enabled": true,
      "surface": "settings",
      "kind": "prompt",
      "priority": 80,
      "title": "Writing prompt",
      "body": "What thought have you been avoiding because it feels too honest?",
      "actionLabel": "Write",
      "action": "compose",
      "maxShows": 5,
      "cooldownHours": 72
    }
  ]
}
```

### 4. Show a known issue

```json
{
  "schema": "thoughts.remote.v1",
  "enabled": true,
  "cards": [
    {
      "id": "known-issue-applinks-001",
      "enabled": true,
      "surface": "settings",
      "kind": "issue",
      "priority": 40,
      "title": "App Links note",
      "body": "Some devices may open shared links in the browser until Android verifies the app link.",
      "actionLabel": "Updates",
      "action": "updates",
      "maxShows": 4,
      "cooldownHours": 96
    }
  ]
}
```

### 5. Show a release highlight

```json
{
  "schema": "thoughts.remote.v1",
  "enabled": true,
  "cards": [
    {
      "id": "release-highlight-150",
      "enabled": true,
      "surface": "settings",
      "kind": "release",
      "priority": 90,
      "title": "App Links are here",
      "body": "Supported shared links can now open Thoughts directly when installed.",
      "actionLabel": "What’s New",
      "action": "whats-new",
      "maxShows": 3,
      "cooldownHours": 72
    }
  ]
}
```

### 6. Show an in-app notice only

```json
{
  "schema": "thoughts.remote.v1",
  "enabled": true,
  "notices": [
    {
      "id": "simple-maintenance-001",
      "enabled": true,
      "priority": 80,
      "message": "Update checks may be slower today. Your notes are not affected.",
      "actionLabel": "",
      "action": "",
      "maxShows": 1,
      "cooldownHours": 168
    }
  ]
}
```

### 7. Show an actionable notice

```json
{
  "schema": "thoughts.remote.v1",
  "enabled": true,
  "notices": [
    {
      "id": "backup-notice-001",
      "enabled": true,
      "priority": 100,
      "message": "Your notes stay local. Export a backup before switching phones.",
      "actionLabel": "Backup",
      "action": "backup",
      "maxShows": 2,
      "cooldownHours": 72
    }
  ]
}
```

### 8. Override guide copy

```json
{
  "schema": "thoughts.remote.v1",
  "enabled": true,
  "copyOverrides": {
    "backupGuideText": "Your notes stay on this device. Export a backup before clearing app data, switching phones, or reinstalling."
  }
}
```

### 9. Disable Share as Image temporarily

```json
{
  "schema": "thoughts.remote.v1",
  "enabled": true,
  "featureFlags": {
    "shareImageEnabled": false
  }
}
```

### 10. Combine card + notice + copy override

```json
{
  "schema": "thoughts.remote.v1",
  "enabled": true,
  "minAppVersion": "1.5.0",
  "lastUpdated": "2026-06-05",
  "featureFlags": {
    "showRemoteCards": true,
    "showRemoteNotices": true,
    "backupImportEnabled": true,
    "shareImageEnabled": true
  },
  "copyOverrides": {
    "backupGuideText": "Your notes stay on this device. Export a backup before switching phones, clearing app data, or reinstalling."
  },
  "cards": [
    {
      "id": "backup-safety-001",
      "enabled": true,
      "surface": "settings",
      "kind": "backup",
      "priority": 100,
      "title": "Protect your notes",
      "body": "Thoughts stores notes locally. Export a backup before switching phones or clearing app data.",
      "actionLabel": "Open Backup",
      "action": "backup",
      "maxShows": 8,
      "cooldownHours": 48
    }
  ],
  "notices": [
    {
      "id": "backup-reminder-remote-001",
      "enabled": true,
      "priority": 100,
      "message": "Your notes stay local. Export a backup before switching phones.",
      "actionLabel": "Backup",
      "action": "backup",
      "maxShows": 2,
      "cooldownHours": 72
    }
  ]
}
```

## Invalid examples

### Invalid: dangerous action

```json
{
  "schema": "thoughts.remote.v1",
  "cards": [
    {
      "id": "bad-001",
      "enabled": true,
      "surface": "settings",
      "kind": "warning",
      "priority": 100,
      "title": "Clean up",
      "body": "Delete everything now.",
      "actionLabel": "Delete",
      "action": "deleteAll",
      "maxShows": 1,
      "cooldownHours": 1
    }
  ]
}
```

The app must reject `deleteAll`.

### Invalid: arbitrary URL

```json
{
  "schema": "thoughts.remote.v1",
  "cards": [
    {
      "id": "bad-url-001",
      "enabled": true,
      "surface": "settings",
      "kind": "info",
      "priority": 10,
      "title": "Open site",
      "body": "This domain is not trusted.",
      "actionLabel": "Open",
      "action": "https://example.com",
      "maxShows": 1,
      "cooldownHours": 24
    }
  ]
}
```

Only trusted domains should be allowed.

### Invalid: too much content

Remote cards are not blog posts. Keep them short. Large text should go into a normal app update, a guide page, or GitHub release notes.

## Operational rules

1. Validate JSON before pushing.
2. Keep old working config backed up.
3. Change one campaign at a time.
4. Use unique IDs for new campaigns.
5. Do not reuse IDs unless updating the same campaign.
6. Keep `maxShows` low.
7. Keep `cooldownHours` high.
8. Never use remote config to manipulate privacy or storage settings.
9. Test with bad JSON.
10. Test offline behavior.
11. Test app start with no config.
12. Test app start with cached config.
13. Keep the file under 120 KB.

## Recommended publishing workflow

1. Edit `remote_config.json`.
2. Validate JSON using a JSON formatter.
3. Check schema is `thoughts.remote.v1`.
4. Confirm actions are safe.
5. Confirm card IDs are unique.
6. Push to GitHub.
7. Open raw URL in browser.
8. Wait for GitHub raw cache if needed.
9. Test app with force refresh if you add that control.
10. Watch for layout issues.

## Version strategy

Use `minAppVersion` when a card depends on new app code.

Example:

```json
"minAppVersion": "1.5.0"
```

For future features requiring v1.6.0:

```json
"minAppVersion": "1.6.0"
```

## Privacy promise

Remote Config only fetches a public JSON file. It must not upload notes, drafts, settings, identifiers, or user content.

Suggested user-facing Settings copy:

```text
Live Content lets Thoughts show helpful notices, safety reminders, guide updates, and release highlights without requiring an app update. Your notes are never sent anywhere.
```

## Final recommendation

Keep Remote Config v1 boring and strict. That is the point.

Use it for:
- helpful cards
- notices
- copy fixes
- safe feature flags
- known issues
- release highlights
- writing prompts

Do not use it for:
- destructive actions
- privacy control
- app lock control
- database changes
- encryption behavior
- arbitrary URLs
- remote code
