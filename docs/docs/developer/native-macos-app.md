# Native macOS app blueprint (monorepo-first)

If you want to build a **native SwiftUI macOS app for Immich**, this guide gives a practical architecture and staged plan that stays **inside the Immich monorepo** and reuses as many existing components and contracts as possible.

## Product goals

For v1, "native" should mean:

- SwiftUI UI and navigation (not a wrapped web view).
- macOS-first UX (keyboard shortcuts, multi-window, menu bar hooks).
- Local-first browsing with fast timeline scrolling.
- Upload/sync from user-selected folders.

Suggested v1 scope:

1. Server setup/login (URL + API key or OAuth when available).
2. Timeline/album browsing with thumbnails.
3. Detail view for photo/video.
4. Desktop uploads (drag-and-drop + watched folder).

## Monorepo-first principles

1. **Keep the app in this repository** under `desktop-macos/`.
2. **Reuse existing API contracts** from `open-api/` instead of manually defining models.
3. **Reuse platform-native code patterns** already used in `mobile/ios` (Keychain, AVFoundation, networking conventions) where applicable.
4. **Reuse shared product behavior** from web/mobile specs (album semantics, timeline sorting, upload states) so users get consistent behavior across clients.

## Proposed monorepo layout

```text
desktop-macos/
  ImmichMacApp/                # SwiftUI app target
  Packages/
    ImmichAPI/                 # generated + wrapped API client from open-api/
    ImmichCore/                # models/use-cases shared across macOS modules
    ImmichPersistence/         # SQLite/GRDB schema + repositories
    ImmichMedia/               # thumbnail/video loading and cache
    ImmichSync/                # upload queue + folder watcher
  ImmichMacApp.xcodeproj
```

## Architecture choices

### 1) App shell

- **Framework:** SwiftUI with `NavigationSplitView`.
- **State:** TCA or a lightweight observable-state architecture.
- **Data layer:** Async/await networking + local cache database.

### 2) API integration (reuse-first)

- Generate a typed Swift client from `open-api/`.
- Keep one internal API module (`ImmichAPI`) to isolate endpoint changes.
- Add retry + cancellation for long-running library queries.

### 3) Local storage

- Use **GRDB/SQLite** for metadata cache (assets, albums, users, sync cursors).
- Store thumbnail/media files under Application Support with deterministic paths.
- Maintain sync cursors per endpoint for incremental refresh.

### 4) Media pipeline

- Thumbnails: prefetch + memory/disk cache (Nuke or equivalent).
- Video playback: AVKit with lazy loading and progressive buffering.
- Metadata extraction for local imports: ImageIO + AVFoundation.

### 5) Sync and uploads

- Folder watch via `FSEvents`.
- Queue uploads with backoff, dedupe, pause/resume, and conflict handling.
- Respect macOS background execution limits and permission prompts.

## Milestone plan

### Milestone 0 — scaffold in monorepo

- Create `desktop-macos/` workspace and CI build on macOS runner.
- Add generated API module from `open-api/`.
- Implement auth/session storage in Keychain.

### Milestone 1 — browsing

- Timeline API integration.
- Adaptive grid with smooth scrolling.
- Asset detail screen (zoom, EXIF, location).

### Milestone 2 — uploads

- Manual upload and drag-and-drop.
- Watched folder sync engine.
- Upload queue UI with retry/cancel.

### Milestone 3 — native macOS polish

- Keyboard shortcuts and multi-window support.
- Quick Look integration.
- Menu bar status and notifications.

## CI and quality gates

- Unit tests for API client and sync engine.
- Snapshot/UI tests for major views.
- Integration tests against local Immich stack.
- Performance checks for large-library scrolling and thumbnail decoding.

## Security and privacy checklist

- Store tokens in Keychain only.
- Never log auth headers or sensitive file paths.
- Respect Files/Folders permissions and provide clear controls for watched paths.

## First week execution checklist

1. Scaffold `desktop-macos/ImmichMacApp` in the monorepo.
2. Generate Swift API client from `open-api/` and wire login flow.
3. Render timeline thumbnails from real server data.
4. Add basic drag-and-drop upload queue.
5. Set up CI build + smoke UI test on a macOS runner.

---

Next step: scaffold a minimal `desktop-macos/` app target in this repository with login + read-only timeline, then iterate by milestones.
