# Thoughts Remote Config v1 Guide

This file documents the live-content format supported by the current `lib/main.dart`.

Remote Config is for safe live content, copy fixes, modal safety notices, and disabling already-built app features. It cannot add new Flutter code, run remote code, read notes, upload data, or perform destructive actions that are not already built into the APK.

## Current Behavior

- The app fetches remote config on startup and app resume.
- App resume uses force refresh so urgent safety changes are checked quickly.
- Normal non-forced refresh cooldown is 10 minutes.
- A successful response is cached for offline fallback.
- `enabled: false` clears remote flags, cards, copy, and notices from app state.
- Remote notices are shown as modal popups, not undo/top notices.
- High-priority notices with `priority >= 100` bypass local show history and cooldown.
- Up to three safe action buttons can be shown in one remote notice.
- Remote config can disable features, but it cannot enable dangerous behavior.

## Minimal Valid File

```json
{
  "schema": "thoughts.remote.v1",
  "enabled": true,
  "minAppVersion": "2.3.0",
  "lastUpdated": "2026-06-09"
}
```

## Supported Feature Flags

All flags are booleans. Missing flags use the app fallback.

```text
showRemoteCards
showRemoteNotices
showKnownIssues
showWritingPrompts
showReleaseHighlight
remoteTipsEnabled
appLinksCardEnabled
launchKeyboardCardEnabled

factoryResetEnabled
dataResetEnabled
minimalModeEnabled
backupImportEnabled
backupExportEnabled
encryptedBackupExportEnabled
testBackupEnabled
shareImageEnabled
appLockEnabled
screenProtectionEnabled
appIconSwitchingEnabled
updateChecksEnabled
importantNoticesEnabled
```

### Standard Disable UX

If a live config disables an active feature, the app should explain it with a modal:

- Minimal Mode is exited and the user is told it was disabled by a live safety notice.
- App Lock is turned off and the user is told it was disabled by a live safety notice.
- Screen Protection is turned off and the user is told it was disabled by a live safety notice.
- Important Notices are turned off and the user is told it was disabled by a live safety notice.

For other disabled features, rows/buttons show unavailable copy and block the action.

## Supported Copy Overrides

```text
backupGuideText
settingsBackupSubtitle
settingsNoticeSubtitle
appLinksHelpText
privacyReminderText
releaseHighlightText
virusTotalVersion
virusTotalApkName
virusTotalHash
virusTotalScore
virusTotalUpdatedAt
virusTotalUrl
```

Copy values are capped by the app. Keep them short, calm, and factual.

## Safe Actions

Remote cards, notices, and notice actions may use:

```text
home
open
settings
privacy
privacy-security
privacy-policy
experience
appearance
accessibility
app-lock
applock
health
diagnostics
updates
update
releases
notifications
notices
about
guides
whats-new
whatsnew
changelog
release-notes
language
languages
language-store
backup
compose
new
note
github
dismiss
```

Trusted HTTPS hosts:

```text
writethoughts.netlify.app
github.com
raw.githubusercontent.com
dheeraz.netlify.app
www.virustotal.com
virustotal.com
```

Do not use arbitrary URLs.

## Remote Cards

Cards are small Settings cards. Keep them short.

```json
{
  "id": "factory-reset-known-issue-230-002",
  "enabled": true,
  "surface": "settings",
  "kind": "warning",
  "priority": 999,
  "title": "Factory Reset is temporarily disabled",
  "body": "Factory Reset has a known issue in older v2.3.0 test builds. Export a backup before clearing app data.",
  "actionLabel": "Open Backup",
  "action": "backup",
  "maxShows": 20,
  "cooldownHours": 6
}
```

Supported surfaces:

```text
settings
home
updates
privacy
backup
```

Supported kinds:

```text
info
warning
backup
prompt
release
issue
```

## Remote Notices

Remote notices are modal popups with OK/Cancel and optional action buttons.

Legacy single-action format is still supported:

```json
{
  "id": "backup-notice-001",
  "enabled": true,
  "title": "Backup reminder",
  "priority": 80,
  "message": "Your notes stay local. Export a backup before switching phones.",
  "actionLabel": "Backup",
  "action": "backup",
  "maxShows": 2,
  "cooldownHours": 72
}
```

New multi-action format:

```json
{
  "id": "factory-reset-warning-230-002",
  "enabled": true,
  "title": "Factory Reset safety notice",
  "priority": 100,
  "message": "Factory Reset is temporarily disabled in this build while a reset issue is being checked. Your notes are safe. Export a backup before clearing app data.",
  "actions": [
    { "label": "Backup", "action": "backup" },
    { "label": "Privacy", "action": "privacy" },
    { "label": "Updates", "action": "updates" }
  ],
  "maxShows": 5,
  "cooldownHours": 1
}
```

Rules:

- `id`: required, stable, under 80 characters.
- `title`: optional, under 70 characters.
- `message`: required, under 320 characters.
- `actions`: optional, max 3 buttons.
- Each action label must be under 32 characters.
- Each action must be safe.
- `priority >= 100` bypasses local show history/cooldown for emergency alerts.
- Use a new ID when you need users to see a materially new notice.

## Emergency Disable Example

```json
{
  "schema": "thoughts.remote.v1",
  "enabled": true,
  "minAppVersion": "2.3.0",
  "lastUpdated": "2026-06-09",
  "featureFlags": {
    "factoryResetEnabled": false,
    "showRemoteNotices": true,
    "showRemoteCards": true
  },
  "notices": [
    {
      "id": "factory-reset-warning-230-002",
      "enabled": true,
      "title": "Factory Reset safety notice",
      "priority": 100,
      "message": "Factory Reset is temporarily disabled in this build while a reset issue is being checked. Export a backup before clearing app data.",
      "actions": [
        { "label": "Backup", "action": "backup" },
        { "label": "Updates", "action": "updates" }
      ],
      "maxShows": 5,
      "cooldownHours": 1
    }
  ]
}
```

## Operational Rules

1. Validate JSON before pushing.
2. Keep a backup of the last working config.
3. Change one campaign at a time.
4. Use unique IDs for new notices/cards.
5. Use `enabled: false` to retire old campaigns.
6. Use `priority >= 100` only for real safety issues.
7. Use feature flags only to disable/hide built-in behavior.
8. Do not use remote config to delete data or run destructive actions.
9. Keep the file under 120 KB.
10. Test app start, app resume, offline cache, and bad JSON.

## Privacy Promise

Remote Config fetches a public JSON file. It must not upload notes, drafts, settings, identifiers, or user content.
