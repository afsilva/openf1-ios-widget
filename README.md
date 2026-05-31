# OpenF1 Dashboard (iOS Widget First: iOS + iPadOS + macOS via iPhone widgets)

This project is now intentionally simplified to a **single active widget path**:

- **Active/maintained:** iOS WidgetKit app + widget extension
- **Runs on:** iPhone, iPad, and macOS (via iPhone widgets on Mac)
- **Obsolete/deprecated:** native macOS widget target path

Data source: [OpenF1 API](https://api.openf1.org)

License: **GPL-3.0-or-later** (`LICENSE`)

---

## Why this simplification

You identified that the iOS widget path is more reliable in practice. To reduce complexity and maintenance burden, this repo now focuses on the widget path that works best across your devices.

That means:
- one primary app/widget target pair,
- one shared service/model path,
- fewer signing, cache, and extension-registration permutations.

---

## Compatibility

- **iOS 17+**
- **iPadOS 17+**
- **macOS (Apple Silicon) via iPhone widgets on Mac**

> Native macOS widget targets are kept in repo history/runtime folders for reference but are no longer generated as active targets from `project.yml`.

---

## Active project layout

```text
openf1-ios-widget/
  XcodeProjectTemplate/
    project.yml                      # iOS-only active targets
    SETUP_CHECKLIST.md
    Scripts/generate_xcodeproj.sh
    Config/*.plist + *.entitlements
    OpenF1DashboardiOSApp/*          # active
    OpenF1DashboardiOSWidget/*       # active
    OpenF1Shared/*                   # active shared code
```

---

## Setup (iOS-first)

```bash
cd openf1-ios-widget/XcodeProjectTemplate
./Scripts/generate_xcodeproj.sh
open OpenF1Dashboard.xcodeproj
```

Then configure only:
- `OpenF1DashboardiOSApp`
- `OpenF1DashboardiOSWidget`

### Required capabilities

Enable the same **App Group** on both iOS targets, e.g.:
- `group.com.yourorg.openf1widget`

Update in code:
- `OpenF1Shared/OpenF1Service.swift`
- `AppGroupConfig.identifier`

---

## Behavior

- Calendar of next relevant sessions (non-canceled)
- Driver + constructor standings from OpenF1 session results
- Cache-first refresh with fallback to last-known-good model
- Manual refresh via widget intent button

Session row order is standardized as:
- **System / Local / UTC**

Refresh policy:
- daily off-weekend
- hourly around race weekends

Version/build convention:
- `CFBundleShortVersionString` (marketing version) uses date format: `YYYY.MM.DD`
- `CFBundleVersion` (build number) increments for same-day rebuilds (`1`, `2`, `3`, ...)
- Widget build-stamp text (if shown in UI) should mirror date + build

---

## Security posture (OWASP-aligned)

Key controls retained:
- fixed HTTPS host (`api.openf1.org`)
- endpoint allowlist
- query validation
- bounded response/cache sizes
- sanitized rendered text
- genericized error/fallback UI
- no secret/token handling in code/cache

See `SECURITY_OWASP_TOP10.md` for assessment details.

---

## Obsolete native macOS widget path

Native macOS app/widget targets were deprecated and removed to keep the repo focused on the iOS-first path.

---

## Related docs

- `PROMPT.md` — updated regeneration prompts
- `SECURITY_OWASP_TOP10.md` — OWASP Top 10 assessment
- `APP_STORE_DEPLOYMENT.md` — deployment guidance
- `XcodeProjectTemplate/SETUP_CHECKLIST.md` — iOS-first setup checklist
