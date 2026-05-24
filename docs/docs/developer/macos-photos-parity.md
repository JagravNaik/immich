# macOS Photos Parity Plan for Immich Desktop

This document treats Apple Photos for macOS as the reference product and `desktop-macos/` as the implementation target.

The goal is not just to match the current Immich web app on macOS. The goal is to build a native desktop client that behaves like a serious Photos.app clone, with Immich replacing iCloud as the storage, sync, and metadata backend.

Companion document:

- Motion and animation parity spec: [`macos-photos-motion-parity.md`](/Users/jagravnaik/Projects/immich/docs/docs/developer/macos-photos-motion-parity.md)
- Apple user-guide feature audit: [`macos-photos-user-guide-audit.md`](/Users/jagravnaik/Projects/immich/docs/docs/developer/macos-photos-user-guide-audit.md)

## Scope and methodology

This document combines:

- Direct inspection of Apple Photos.app on macOS
- Inspection of the current Immich macOS app code in `desktop-macos/`
- A review of the current API surface exposed by `ImmichAPIClient`
- A practical parity framing: what can be implemented entirely in the app, what requires backend work, and what requires deliberate product decisions

This is a product and systems design document, not just a UI checklist.

## Executive summary

The Immich macOS app is already much farther along than a typical prototype.

It already has:

- Native SwiftUI shell and authentication flow
- A Photos-inspired sidebar and split-view layout
- Timeline browsing with `Years`, `Months`, and `All Photos`
- Full-screen viewer with hero transitions
- Live Photo playback support
- Map browser
- Collections, albums, people, memories, screenshots, favorites, videos, panoramas, and trash routes
- Search with multiple search modes
- Multi-select actions
- Asset info inspector
- A local editing pipeline
- Import, download, share, tag, album, API key, and admin-user flows

But it is not yet close to full Apple Photos parity.

The biggest gaps are:

1. Product architecture parity
   Apple Photos is a deeply layered browser, organizer, viewer, editor, and sync client. Immich already covers parts of all of those, but not with the same depth or cohesion.

2. Editing parity
   Immich has a custom editor, but Apple Photos' editor is much richer, more structured, more discoverable, and more nondestructive in feel.

3. Search and semantic browsing parity
   Apple Photos exposes people, groups, places, recency, media types, and memory surfaces as first-class product concepts. Immich has some of this, but the behavior is less complete and less integrated.

4. macOS-native workflow parity
   Apple Photos is not just a gallery. It is a desktop-native media management tool with strong import, selection, inspection, sharing, keyboard, and editing workflows. Immich still needs more AppKit/macOS-specific depth here.

5. Backend parity prerequisites
   Some Apple Photos behaviors depend on data and capabilities the current Immich backend either does not expose or does not model in the same way. Full parity is therefore not only a client project.

If the real target is "Photos.app, but backed by Immich", the app should be designed as a native media workstation, not just a SwiftUI shell over existing endpoints.

## Apple Photos: product model

Apple Photos on macOS is best understood as six systems working together:

1. Archive browser
   A dense, high-performance chronological library for everything in the collection.

2. Semantic browser
   Alternate ways to traverse the same collection by people, places, media types, favorites, imports, screenshots, deleted items, and memory groupings.

3. Viewer
   A focused single-item experience with quick metadata access, neighboring-item navigation, and transitions back to the source context.

4. Editor
   A dedicated nondestructive workspace for adjustments, styles, crop, cleanup, enhancement, and extension-based editing.

5. Organizer
   Albums, favorites, shared surfaces, curation, deletion, restore, metadata correction, and selection workflows.

6. Sync and system integration layer
   iCloud sync, local caching, import, export, search, sharing, system tool integration, and background maintenance.

Apple Photos is not designed as a collection of unrelated screens. It is designed around one media graph that can be viewed through many lenses.

## Apple Photos: information architecture

Observed sidebar structure:

- `Library`
- `Collections`
- `Pinned`
  - `Favorites`
  - `Recently Saved`
  - `Map`
  - `Videos`
  - `Screenshots`
  - `People`
  - `Recently Deleted`
- `Albums`
- `Sharing`
  - `Shared Albums`
  - `Activity`
  - `Shared with You`
- `Media Types`
- `Utilities`
- `Projects`

Important design implication:

- Apple separates "the entire archive" from "curated discovery surfaces".
- Apple also separates "semantic slices of the library" from "user-authored organization" like albums.
- The sidebar is both navigation and product taxonomy. It teaches the user what the system understands about the library.

## Apple Photos: screen-by-screen reference

### 1. Library

Observed behaviors:

- Main archive grid
- Time-grouped browsing
- Top toolbar mode switch: `Years`, `Months`, `All Photos`
- Adjustable grid density via zoom controls
- Search in the global toolbar
- Selection-sensitive toolbar actions
- Footer shows library totals and sync status

Design characteristics:

- Very dense
- Very fast visual scanning
- Minimal text in the grid itself
- Strong emphasis on chronology
- Controls are quiet unless context demands otherwise

Key parity requirements:

- Huge-library performance
- Smooth density transitions
- Strong keyboard navigation
- Stable selection and source context
- Sync and loading states that do not break navigation

### 2. Collections

Observed behaviors:

- Card-based discovery surface
- `Memories`
- `Pinned`
- `Albums`
- `People`
- `Featured Photos`

Design characteristics:

- Less utilitarian than `Library`
- Larger cards, more editorial presentation
- Designed to surface recency, meaning, and delight

Key parity requirements:

- A dedicated discovery surface, not just another list
- Memory cards with strong cover treatment
- Pinned utilities and pinned user collections
- People and albums displayed as first-class browse objects

### 3. People

Observed behaviors:

- Separate `People` view
- Grouped social clusters
- Individual person entries
- Sorting
- Large face-forward cards

Design characteristics:

- The app treats people recognition as a core browsing system
- Both individuals and recurring combinations are surfaced

Key parity requirements:

- Person entities
- Person groupings or "people together" surfaces
- Hidden/merged/named people workflows
- Fast transition from people browse to person-specific timeline

### 4. Map

Observed behaviors:

- Geographic browse mode
- Clustered photo thumbnails on the map
- `Map`, `Satellite`, and `Grid` modes
- Zoomable map canvas

Design characteristics:

- The map is not a novelty view
- It is a full browsing mode with a dedicated control scheme

Key parity requirements:

- Marker clustering
- Place search
- Place-focused browsing
- Alternate map display modes
- Fast transition from map cluster to local gallery

### 5. Single-item viewer

Observed behaviors:

- Opening an item changes the toolbar and interaction model
- Large centered media canvas
- Back navigation
- Date and location context in the toolbar
- Favorite, share, rotate, auto-enhance, info, and edit actions
- Selection index in the current context

Design characteristics:

- The viewer is still contextualized inside the library
- It feels modal in focus, but not disconnected from source context

Key parity requirements:

- Source-aware entry and exit
- Fast paging through neighbors
- Stable toolbar state
- Rich metadata and related actions

### 6. Info / inspector

Observed behaviors:

- Info is treated as a first-class asset state
- Metadata includes filename, date/time, size, dimensions, camera/exif, and location
- Location can include an embedded map
- Recognition/semantic overlays can appear on-image

Key parity requirements:

- Dedicated inspector model
- Fast metadata fetch
- Good empty/loading/error states
- Location, camera, descriptive metadata, and derived metadata

### 7. Editing

Observed behaviors:

- Separate edit workspace entered from viewer
- Tabbed tools: `Adjust`, `Styles`, `Crop`, `Clean Up`
- Strong right-hand editing inspector
- Many expandable adjustment groups
- Auto and revert affordances
- Editing actions are separate from browsing actions

Design characteristics:

- Editing is deep, but still approachable
- It is structured, hierarchical, and discoverable
- The editor feels nondestructive and stateful

Key parity requirements:

- Dedicated editing mode
- Nontrivial adjustment graph
- Styles/filters
- Crop/straighten/rotate/flip
- Cleanup/removal tools
- Nondestructive session model
- Clear save/revert model

### 8. Sharing, albums, and management

Observed behaviors:

- Albums are distinct from browse modes
- Sharing is a dedicated product area
- Recently deleted is treated as a recoverable state, not instant destruction
- Toolbar and context-menu actions support common desktop workflows

Key parity requirements:

- Album CRUD and asset membership management
- Share/export flows
- Recoverable trash
- Bulk operations
- Desktop-native open/save/share affordances

## Apple Photos: interaction patterns that matter

The highest-value patterns to preserve in an Immich clone are:

### A. One data model, many browse lenses

The same assets appear in library, map, people, albums, favorites, screenshots, memories, and deleted views. These are not separate data silos.

### B. Selection sensitivity

The app changes available actions depending on whether the user is browsing, has a single item selected, or is editing a single item.

### C. Source-context preservation

When you open a photo from a browse context, it still belongs to that context. Paging moves through neighboring items from the active context.

### D. Progressive disclosure

Toolbar controls and edit tools stay quiet until needed.

### E. Visual-first organization

Apple minimizes text-heavy management UIs and keeps the product image-led.

### F. Strong mode boundaries

Browse mode, view mode, info mode, and edit mode are distinct enough that users rarely feel lost.

## Current Immich macOS app: implementation inventory

The current app lives in `desktop-macos/` and already implements a broad Photos-inspired shell.

### App shell and architecture

Implemented:

- Native SwiftUI app shell in [`desktop-macos/Sources/ImmichMacApp/ImmichMacApp.swift`](/Users/jagravnaik/Projects/immich/desktop-macos/Sources/ImmichMacApp/ImmichMacApp.swift)
- Main split-view application shell in [`desktop-macos/Sources/ImmichMacApp/Views/MainContentView.swift`](/Users/jagravnaik/Projects/immich/desktop-macos/Sources/ImmichMacApp/Views/MainContentView.swift)
- Sidebar destination model in [`desktop-macos/Sources/ImmichMacApp/Navigation/SidebarView.swift`](/Users/jagravnaik/Projects/immich/desktop-macos/Sources/ImmichMacApp/Navigation/SidebarView.swift)
- App-wide state machine and view state in [`desktop-macos/Sources/ImmichMacApp/State/AppState.swift`](/Users/jagravnaik/Projects/immich/desktop-macos/Sources/ImmichMacApp/State/AppState.swift)

### Authentication and sessions

Implemented:

- Server setup
- Password login
- API key login
- OAuth flow
- Session resume
- Keychain-backed credential storage

### Library browsing

Implemented:

- `Library`
- Timeline buckets
- `Years`, `Months`, `All Photos`
- Adaptive grid density
- Hero transitions from grid to viewer
- Keyboard shortcuts
- Multi-select
- Drag and drop import into the grid

Relevant files:

- [`LibraryGridView.swift`](/Users/jagravnaik/Projects/immich/desktop-macos/Sources/ImmichMacApp/Views/LibraryGridView.swift)
- [`MainContentView.swift`](/Users/jagravnaik/Projects/immich/desktop-macos/Sources/ImmichMacApp/Views/MainContentView.swift)
- [`AppState.swift`](/Users/jagravnaik/Projects/immich/desktop-macos/Sources/ImmichMacApp/State/AppState.swift)

### Collections and semantic browsing

Implemented:

- `Collections`
- `Map`
- `All Albums`
- `Album`
- `All People`
- `Person`
- `All Memories`
- `Memory`
- `Videos`
- `Live Photos`
- `Panoramas`
- `Screenshots`
- `Favorites`
- `Imports`
- `Recently Deleted`

Relevant files:

- [`SidebarView.swift`](/Users/jagravnaik/Projects/immich/desktop-macos/Sources/ImmichMacApp/Navigation/SidebarView.swift)
- [`CollectionsView.swift`](/Users/jagravnaik/Projects/immich/desktop-macos/Sources/ImmichMacApp/Views/CollectionsView.swift)
- [`MapBrowserView.swift`](/Users/jagravnaik/Projects/immich/desktop-macos/Sources/ImmichMacApp/Views/MapBrowserView.swift)

### Viewer and asset inspection

Implemented:

- Full viewer
- Hero animation
- Neighbor paging
- Live Photo playback
- Video playback
- Panorama viewer
- External info inspector panel
- Tag editing entry points
- Download, share, trash, favorite actions

Relevant files:

- [`PhotoDetailView.swift`](/Users/jagravnaik/Projects/immich/desktop-macos/Sources/ImmichMacApp/Views/PhotoDetailView.swift)
- [`AssetInfoInspector.swift`](/Users/jagravnaik/Projects/immich/desktop-macos/Sources/ImmichMacApp/Views/AssetInfoInspector.swift)
- [`AssetInfoPanelController.swift`](/Users/jagravnaik/Projects/immich/desktop-macos/Sources/ImmichMacApp/Views/AssetInfoPanelController.swift)

### Search

Implemented:

- Expand/collapse search field
- Recent searches overlay
- Search modes: smart, filename, description, OCR
- Backend-driven paged search
- Filter data model

Relevant files:

- [`ToolbarSearchField.swift`](/Users/jagravnaik/Projects/immich/desktop-macos/Sources/ImmichMacApp/Views/ToolbarSearchField.swift)
- [`ImmichModels.swift`](/Users/jagravnaik/Projects/immich/desktop-macos/Sources/ImmichCore/ImmichModels.swift)
- [`AppState.swift`](/Users/jagravnaik/Projects/immich/desktop-macos/Sources/ImmichMacApp/State/AppState.swift)

### Editing

Implemented:

- Editing sidebar
- Tabs: `Adjust`, `Filters`, `Crop`
- Local Core Image editing pipeline
- Auto enhance
- Filter thumbnails
- Straighten/rotate/flip
- Save edited image back to server
- Export edited image to disk

Relevant files:

- [`EditingSidebar.swift`](/Users/jagravnaik/Projects/immich/desktop-macos/Sources/ImmichMacApp/Views/EditingSidebar.swift)
- [`PhotoEditingPipeline.swift`](/Users/jagravnaik/Projects/immich/desktop-macos/Sources/ImmichMacApp/State/PhotoEditingPipeline.swift)

### Organization and management

Implemented:

- Album create, rename, delete
- Add/remove assets to/from albums
- Tags CRUD and tag assignment
- API key management
- Admin user management
- Trash and restore
- Bulk favorite/download/tag/trash actions

Relevant files:

- [`ManagementSheets.swift`](/Users/jagravnaik/Projects/immich/desktop-macos/Sources/ImmichMacApp/Views/ManagementSheets.swift)
- [`AppState.swift`](/Users/jagravnaik/Projects/immich/desktop-macos/Sources/ImmichMacApp/State/AppState.swift)

### Upload/import

Implemented:

- File import from `NSOpenPanel`
- Drag-and-drop import
- Upload queue state actor
- Upload progress and failure banner

Relevant files:

- [`UploadQueue.swift`](/Users/jagravnaik/Projects/immich/desktop-macos/Sources/ImmichSync/UploadQueue.swift)
- [`AppState.swift`](/Users/jagravnaik/Projects/immich/desktop-macos/Sources/ImmichMacApp/State/AppState.swift)

## Parity assessment

The app is best described as:

- Strong browser/viewer foundation
- Medium semantic-browsing coverage
- Early-to-mid editing implementation
- Good Immich-specific management tooling
- Not yet a full Photos.app clone

## Major parity gaps

### 1. Sidebar and information architecture are only partially aligned

Current Immich strengths:

- Top-level `Library`, `Collections`, `Map`
- Media types and utility routes
- Albums and people support

Current parity gaps:

- No dedicated `Pinned` system matching Apple's pinned browse shortcuts
- No `Recently Saved` route
- No `Sharing` area matching Apple Photos' shared albums/activity/shared-with-you grouping
- No `Projects` equivalent
- No clear separation between semantic browse destinations and user/library management destinations

Recommended work:

- Rebuild the sidebar taxonomy to match Apple Photos more closely
- Introduce `Pinned` as a first-class section with system items and user-pinned items
- Add a proper `Sharing` cluster if the backend supports it
- Keep Immich-only management tools available, but move them out of the primary photo-browsing IA

### 2. Collections is present, but not yet equivalent

Current Immich strengths:

- Dedicated `CollectionsView`
- People, memories, albums, and media-type quick access cards

Current parity gaps:

- No `Featured Photos`
- No strong `Pinned` system inside Collections
- No people groups / social clusters equivalent
- Albums and memories are visually simpler than Apple's card treatment
- Lacks the same editorial/discovery depth

Recommended work:

- Add richer cover-card design and more shelf types
- Add a `Pinned` shelf
- Add people groups if backend data can support it
- Add recency and highlights curation beyond current memories/albums sections

### 3. Library view parity is close in layout, but not yet in behavior

Current Immich strengths:

- Timeline modes
- Adaptive density
- Keyboard support
- Selection model
- Hero transitions

Current parity gaps:

- More-options menu has explicit TODOs for important filters and sort modes in [`MainContentView.swift`](/Users/jagravnaik/Projects/immich/desktop-macos/Sources/ImmichMacApp/Views/MainContentView.swift)
- No completed `hide screenshots`, `photos only`, `videos only`, `sort by date captured`, or `sort by date added` implementation
- No visible footer equivalent for sync/library counts in the browser surface
- Apple's visual grouping and empty/loading semantics are more polished

Recommended work:

- Implement the currently stubbed browser filter/sort actions first
- Add richer footer/status treatment
- Tighten browse-state persistence and route-specific filtering behavior

### 4. Viewer parity is good, but inspector parity is only medium

Current Immich strengths:

- Full-screen viewer
- Paging
- Favorite/share/download/trash/tag actions
- Live Photo and video playback
- Panorama handling

Current parity gaps:

- Apple's viewer toolbar is more tightly integrated with browse context and edit state
- Apple exposes more polished item indexing/context information
- Apple's info model includes stronger semantic overlays and object/recognition cues
- Immich uses a separate floating panel model for info, which may or may not match the intended clone experience

Recommended work:

- Decide whether the clone should use a floating inspector or an integrated right inspector like modern Photos edit mode
- Enrich asset metadata surface with more semantic detail where backend data exists
- Tighten transitions between viewer, info, and edit

### 5. Editing parity is the single largest client-side gap

Current Immich strengths:

- Real editing pipeline
- Adjustments
- Filters
- Straighten/rotate/flip
- Save back to server
- Export to disk

Current parity gaps:

- No `Clean Up` tool
- No full Apple-style `Styles` system
- No deep adjustment taxonomy comparable to Photos
- `Crop` is explicitly incomplete in the UI: `Aspect Ratio` is marked `Coming soon` in [`EditingSidebar.swift`](/Users/jagravnaik/Projects/immich/desktop-macos/Sources/ImmichMacApp/Views/EditingSidebar.swift)
- No visible crop rectangle editor in the viewer UI even though the pipeline has `cropRect` and `cropAspectRatio`
- No nondestructive edit history/version UX
- No plugin/extensions model
- No red-eye, curves, levels, selective color, noise reduction, vignette UI parity beyond a much smaller subset
- No portrait-lighting-style parity

Recommended work:

- Reframe editing as a major product track, not a polish task
- Build a proper crop interaction surface on-canvas
- Expand the adjustment model substantially
- Separate `Styles` from generic filters and design it as a curated high-level system
- Decide whether edits are server-destructive, versioned, or sidecar-based
- Introduce edit history/revert/originals concepts

### 6. Search is better than expected, but still behind Photos.app semantics

Current Immich strengths:

- Multiple search modes
- Search filters model
- OCR and description search support

Current parity gaps:

- Apple Photos search feels more semantic and integrated into browse flows
- Search suggestions are currently just recent searches, not rich semantic suggestions
- No obvious saved searches or deep filter-builder UI
- Search filters exist in the model but are not surfaced as a mature UI system

Recommended work:

- Add rich suggestion chips and semantic search affordances
- Add visible filter builder UI
- Add route-aware search behavior

### 7. People parity is partial

Current Immich strengths:

- People browse view
- Person detail view
- Hidden people support in the data model

Current parity gaps:

- No social groups / multi-person clusters like Apple Photos `Groups`
- No obvious merge/rename/confirm identity management flow surfaced in the current UI
- No "People & Pets" equivalent framing

Recommended work:

- Add person-management tools
- Add grouped co-occurrence surfaces if backend can supply them
- Expand people browse beyond a simple grid

### 8. Map parity is promising, but not complete

Current Immich strengths:

- Dedicated map browser
- Marker aggregation
- Multiple map display modes
- Search bar
- Location-specific gallery overlay

Current parity gaps:

- Need to verify behavior at large-library scale
- Apple's grid/map toggle and clustering polish are more refined
- Place hierarchy and place summaries appear more mature in Photos

Recommended work:

- Stress-test marker clustering and viewport logic
- Add richer place summaries and transitions
- Tighten map-to-gallery continuity

### 9. Album and sharing parity are incomplete

Current Immich strengths:

- Album CRUD
- Membership editing
- Pinned albums

Current parity gaps:

- No dedicated shared-album product area in the macOS client IA
- No Apple-style `Activity` / `Shared with You` structure
- Album presentation is simpler than Photos

Recommended work:

- Decide whether shared albums are in scope for true parity
- If yes, treat sharing as a product area, not a small submenu feature

### 10. Import and desktop workflow parity are incomplete

Current Immich strengths:

- File import
- Drag-and-drop import
- Download/export/share

Current parity gaps:

- No watched-folder or folder-sync UX
- No visible background import manager comparable to a desktop-native ingest workflow
- No Quick Look integration
- No explicit multi-window architecture
- No menu bar utility or desktop-native monitoring surface
- No device/camera import workflow equivalent

Recommended work:

- Build watched-folder sync as a first-class feature
- Add import session UI and history
- Add Quick Look support
- Add multi-window support for compare/edit/browse workflows

### 11. Sync and backend parity are not just client problems

To truly clone Photos.app, the backend must support more than generic asset retrieval.

Immich backend/data requirements for parity include:

- Rich person management primitives
- Memory/highlight generation that supports desktop browse quality
- Strong map/place metadata
- Search suggestion and semantic browse support
- Nondestructive edit/version support if Photos-like edit semantics are desired
- Shared album and activity models
- Recently saved/imported semantics
- Potential on-device or hybrid ML strategies for low-latency semantic UX

Without those, the client can mimic the shell but not the full product behavior.

## What is already missing in the current implementation, concretely

These are the clearest "not done" items visible in the current codebase:

### Browser filters and sorts

Explicit TODOs exist for:

- Hide screenshots
- Show only photos
- Show only videos
- Sort by date captured
- Sort by date added

Location:

- [`desktop-macos/Sources/ImmichMacApp/Views/MainContentView.swift`](/Users/jagravnaik/Projects/immich/desktop-macos/Sources/ImmichMacApp/Views/MainContentView.swift)

### Crop UI completeness

The editing pipeline supports crop state, but the editing UI still says `Coming soon` for aspect ratio handling and does not expose a full crop interaction model.

Location:

- [`desktop-macos/Sources/ImmichMacApp/Views/EditingSidebar.swift`](/Users/jagravnaik/Projects/immich/desktop-macos/Sources/ImmichMacApp/Views/EditingSidebar.swift)
- [`desktop-macos/Sources/ImmichMacApp/State/PhotoEditingPipeline.swift`](/Users/jagravnaik/Projects/immich/desktop-macos/Sources/ImmichMacApp/State/PhotoEditingPipeline.swift)

### Watched-folder sync

The current app has import/upload flows, but not a proper watched-folder system matching the roadmap described in the earlier native-macOS blueprint.

### Photos-style sharing IA

The app exposes albums and sharing actions, but not an Apple-style `Sharing` browse area.

### Deep edit parity

No current parity for:

- Clean Up
- Full Styles system
- Full adjustment set
- Nondestructive history/version UX
- Extensions

### Rich people management

No current parity for:

- Grouped people clusters
- Identity correction workflows comparable to Photos
- Pets support framing

## Recommended parity roadmap

If the target is serious parity, the work should be staged as product systems, not isolated tickets.

### Phase 1: Finish the current browser foundation

Build next:

- Complete browser filter/sort TODOs
- Add stronger footer/library status treatment
- Harden route-specific item-count and selection behavior
- Tighten map and collections polish

Why first:

- The browser is the foundation for every other workflow.

### Phase 2: Upgrade information architecture to true Photos-style parity

Build next:

- Rework sidebar taxonomy
- Add `Pinned`
- Add `Recently Saved`
- Add a proper `Sharing` area if backend support exists
- Expand Collections shelves

Why second:

- This aligns the product model before deeper feature growth.

### Phase 3: Rebuild editing as a full subsystem

Build next:

- Proper crop canvas
- Better styles model
- Expanded adjustments
- Revert/original/version semantics
- Cleaner separation between viewer and editor

Why third:

- Editing is the biggest parity gap that users will immediately notice.

### Phase 4: Deep semantic browsing

Build next:

- Better people management
- People groups
- Better memories/highlights
- Better search suggestions and filter UI
- Richer place browsing

Why fourth:

- This closes the gap between "good gallery" and "Photos clone".

### Phase 5: Desktop-native workflows

Build next:

- Watched folders
- Quick Look
- Import manager/history
- Better system notifications
- Multi-window workflows

Why fifth:

- This is where the app stops feeling like a mobile client brought to the desktop.

## Suggested definition of parity

Do not define parity as "the same sidebar labels."

Define parity as:

1. The same primary user journeys exist.
2. The app supports the same mental model:
   one library, many semantic views, strong single-item viewing, serious editing, recoverable organization, and desktop-native ingest/export.
3. Equivalent actions are available in equivalent contexts.
4. Large-library browsing remains fluid.
5. The product feels native to macOS rather than merely available on macOS.

## Final recommendation

Treat the current Immich macOS app as a strong v0.5, not a near-complete Photos clone.

It already has enough product surface to justify a dedicated parity program. But reaching true Photos.app equivalence will require:

- A UI/UX redesign pass around Apple Photos as a system
- A larger editing investment
- Better semantic browsing depth
- More macOS-native workflows
- Backend work for data and behavior parity

If the team accepts that framing, the right next step is to turn this document into a milestone-based implementation spec and start with Phase 1 and Phase 2 in parallel.

Motion should be treated as part of that implementation spec, not as final polish. The app already has enough motion infrastructure that the right next move is to standardize and retune it around a Photos-like motion system rather than continue adding ad hoc animation timings.

## Appendix: test status

`swift test` in `desktop-macos/` currently fails before test execution because `MockImmichAPIClient` in `desktop-macos/Tests/ImmichMacAppTests/AppStateTests.swift` does not implement the newer `resumeSession(server:accessToken:)` requirement from `ImmichAPIClient`.
