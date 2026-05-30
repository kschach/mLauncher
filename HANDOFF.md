# Handoff: PWA App Drawer Feature

## Status: Abandoned / Superseded

This branch added PWA/web shortcut support to the app drawer, but it is based on an
old version of the codebase and **cannot be merged into current main**. See the
"Why this branch is orphaned" section below before deciding whether to continue here.

---

## What Was Built

Three types of web shortcuts were added to the app drawer:

### 1. WebAPKs (auto-detected)
Apps installed via Chrome's "Add to Home Screen" that Android wraps as a real APK.
Detected by package name prefix `org.chromium.webapk.*` during the app list scan in
`getAppsList()`. No user action needed — they appear automatically.

### 2. Chrome pinned shortcuts
PWAs pinned from Chrome via `LauncherApps.getShortcuts()`. The app list scan queries
Chrome and its variants (`com.android.chrome`, `com.chrome.beta`, etc.) for pinned
shortcuts and surfaces them in the drawer. Launched via
`LauncherApps.startShortcut()`.

### 3. Manual URL shortcuts
User-added via a new "Add web shortcut" button in the app drawer. Stored in prefs as
`"Label||https://url"` strings. Launched via `ACTION_VIEW` intent with
`Browser.EXTRA_APPLICATION_ID` for Chrome tab reuse.

---

## Files Changed (all in old package `com.github.droidworksstudio.mlauncher`)

| File | What changed |
|---|---|
| `data/AppListItem.kt` | Added `AppType` enum (`REGULAR`, `WEBAPK`, `SHORTCUT`, `URL_SHORTCUT`), `appType`, `shortcutId`, `pwaUrl` fields |
| `data/Prefs.kt` | Added `pwaUrlShortcuts: MutableSet<String>` preference |
| `MainViewModel.kt` | Added `dispatchAppLaunch()`, `launchPinnedShortcut()`, `launchUrlShortcut()`; extended `getAppsList()` to load all three PWA types |
| `ui/AppDrawerAdapter.kt` | PWA badge/icon rendering; delete handler for `URL_SHORTCUT` type |
| `ui/AppDrawerFragment.kt` | "Add web shortcut" button + dialog; delete listener that removes from prefs |
| `res/layout/fragment_app_drawer.xml` | "Add web shortcut" button |
| `res/values/strings.xml` | Strings for the new button and dialog |
| `common/ContextExtensions.kt` | Fixed web-open helper to use `ACTION_VIEW` instead of `getLaunchIntentForPackage` |

---

## Why This Branch Is Orphaned

During this session (May 2026) we discovered:

- The upstream (`CodeWorksCreativeHub/mLauncher`) has moved **583 commits ahead** of
  this branch's divergence point (`4783d6df`)
- The package was renamed: `droidworksstudio` → `codeworkscreativehub`
- This branch's changes are all in the old package — there is no clean rebase path
- WebAPKs are reportedly already working in the upstream, making part of this feature
  redundant

---

## If You Want to Revive This Work

1. Start a new branch off current `main` (which uses `codeworkscreativehub` package)
2. Port only the pieces still relevant (likely just manual URL shortcuts and Chrome
   pinned shortcuts — verify WebAPK detection first)
3. The logic in `MainViewModel.kt` on this branch is the reference implementation;
   adapt it to the new package structure

---

## Known Bug (unrelated to PWA feature)

Also diagnosed in this session — see branch `fix/recent-apps-not-showing` for a full
handoff. Short version:

- Recent apps don't appear in the normal app drawer (stale cache at cold start)
- Recent apps incorrectly appear in the Hidden Apps view (missing `includeRecentApps
  = false` in `getHiddenApps()`)

Both are one-line fixes in `MainViewModel.kt` and `AppDrawerFragment.kt`.
