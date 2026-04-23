# macOS Photos Motion Parity Plan for Immich Desktop

This document focuses on one specific requirement for a Photos.app-quality clone: motion.

Apple Photos does not feel polished because it has flashy animation. It feels polished because motion is restrained, spatially consistent, and almost always in service of context preservation. The transition itself rarely asks for attention. It helps the user keep their bearings.

This document combines:

- Direct hands-on inspection of Apple Photos.app on macOS
- Review of the current Immich macOS implementation in `desktop-macos/`
- A motion-systems framing: what the user experiences, what the current code does, and what must change for parity

This is a product motion spec, a code audit, and an implementation plan.

## Executive summary

The current Immich macOS app already has meaningful motion work in place.

It already includes:

- Hero-style open and close transitions between the grid and viewer
- Spring-based horizontal paging in the viewer
- Interactive vertical and pinch dismiss behavior
- Grid density and timeline-mode transitions
- Hover, selection, and search-field animations
- Edit-sidebar and overlay transitions

That is a strong foundation.

But it is not yet Photos-quality motion.

The main gap is not the absence of animation. The gap is inconsistency.

Right now, the app uses many local timings and curves across the browsing shell, grid, viewer, search field, and editor. The motion language is therefore fragmented:

- Different parts of the app use slightly different timing for structurally similar events
- Some transitions feel component-level rather than system-level
- The viewer hero transition is good, but the surrounding state changes are still partially orchestrated with hard-coded delays
- The edit flow does not yet carry the same quiet, confident mode shift that Apple Photos has
- Search and sidebar transitions feel like animated widgets rather than structural changes in the workspace

If the target is "Immich with Photos.app quality", motion needs to become a first-class subsystem with shared rules, tokens, and transition ownership.

## What Apple Photos motion feels like

The best high-level description is:

- Calm
- Fast
- Context-preserving
- Layered, but not busy
- Responsive to gesture progress
- Symmetrical on enter and exit

The app almost never animates just because it can. It animates when one of four things happens:

1. The user is changing browsing context
2. The user is focusing one asset
3. The user is entering a stronger editing or inspection mode
4. The user is performing a direct-manipulation gesture like paging, zooming, dragging, or dismissing

Everything else is subtle polish.

## Observed Apple Photos motion principles

### 1. Source continuity is more important than spectacle

When opening an item from the grid, the user understands where it came from.

The item does not simply appear in a modal. It expands out of a source position into a focused viewer state while the rest of the interface recedes. The same is true in reverse when closing.

This is the single most important motion behavior to preserve.

### 2. Most motion is short and settled quickly

Apple Photos rarely lingers.

The app generally feels like:

- quick response on tap/click
- short structural motion
- no over-bounce
- very fast settle after interaction

The result is that the user perceives responsiveness rather than animation duration.

### 3. Gesture-driven motion is continuous, not staged

In the viewer, vertical dismiss and paging behaviors feel directly tied to user input.

The image, backdrop opacity, and scale all appear to move as one interaction. The user feels resistance and release rather than a pre-scripted animation firing after the fact.

### 4. Structural transitions happen in layers

When Apple Photos changes mode, several things change together:

- the toolbar state
- the available actions
- the canvas prominence
- the side panel or edit inspector
- sometimes the background emphasis

But these changes are coordinated so they read as one transition, not many unrelated ones.

### 5. Background surfaces often fade or recede instead of moving dramatically

Photos avoids throwing the whole window around. It prefers:

- dimming
- slight fades
- content replacement with stable framing
- sidebar or inspector reveals that feel attached to the window structure

### 6. Hover and selection polish are present, but not loud

Grid cells, buttons, and controls do respond, but lightly.

The app does not feel like a web dashboard full of animated affordances. It feels like a dense, serious workspace.

## Screen-by-screen Apple motion reference

### 1. Library browsing

Observed behavior:

- Scrolling feels inertial and steady with no gratuitous cell choreography
- Switching between `Years`, `Months`, and `All Photos` preserves chronology as the primary anchor
- Zooming grid density feels like re-layout, not a new screen
- Selection changes are immediate and quiet

Design interpretation:

- The grid itself is mostly stable
- The motion budget is spent on preserving spatial understanding
- There is little or no decorative stagger or card entrance behavior

Parity implication for Immich:

- Avoid elaborate per-cell transitions in the library
- Animate density and timeline changes around a visible anchor
- Keep hover and selection effects understated

### 2. Opening a photo

Observed behavior:

- The chosen item expands into focus
- The dark viewer backdrop arrives with the transition rather than after it
- Toolbar content shifts from browse controls to viewer controls without feeling like a hard cut
- The transition is fast and reads as one coherent movement

Design interpretation:

- This is not a full-screen modal animation
- It is a contextual focus transition

Parity implication for Immich:

- The hero transition should be the backbone of the viewer experience
- Browser fade, toolbar swap, and hero expansion should be controlled together
- Open and close should be more obviously symmetric

### 3. Closing a photo

Observed behavior:

- The viewer collapses back into its source context
- If the user dismisses with gesture momentum, the animation still resolves into the source grid
- The browser background returns in sync with the image collapse

Design interpretation:

- Reverse continuity matters as much as forward continuity
- This is where weak gallery clones usually break immersion

Parity implication for Immich:

- Closing should always prefer a source-aware destination when available
- Fallback fades should exist, but they should feel like exception paths

### 4. Paging between neighboring items

Observed behavior:

- Horizontal movement is crisp and direct
- The transition settles quickly
- The user never loses the sense that they are moving within a contextual sequence

Design interpretation:

- Paging is about continuity inside a focused mode, not about theatrical movement

Parity implication for Immich:

- Maintain fast spring-based paging
- Avoid large damping changes or long settle times
- Keep adjacent-item preloading aligned with motion so paging remains responsive

### 5. Entering edit mode

Observed behavior:

- The viewer becomes an editor without feeling like a separate application
- The right-side tool surface changes structure
- Tool tabs replace browse actions
- The image remains the center of gravity

Design interpretation:

- Entering edit mode is a mode shift, not a page navigation
- The window remains stable while responsibility shifts from browsing to adjustment

Parity implication for Immich:

- Edit mode should not feel like "show a sidebar"
- The canvas, toolbar, and inspector need one coordinated edit-entry transition

### 6. Switching edit tabs

Observed behavior:

- `Adjust`, `Styles`, `Crop`, and `Clean Up` feel like submodes of one workspace
- The active tool area changes smoothly, but not dramatically
- The user maintains orientation because the canvas stays stable

Design interpretation:

- Local tool motion should be smaller than mode-entry motion
- The image should remain the dominant anchor

Parity implication for Immich:

- Use small, consistent transitions within the editor
- Keep tool-content animation subordinate to the photo itself

### 7. Crop mode

Observed behavior:

- The crop canvas and handles appear as editing affordances on the same image
- Controls for straighten, perspective, and ratio feel attached to this editing state

Design interpretation:

- Crop is a canvas-state transition, not just a list of sliders

Parity implication for Immich:

- Full parity requires crop overlays and aspect-ratio affordances that animate into the canvas state
- A "coming soon" placeholder is not near parity

### 8. Collections, People, and Map

Observed behavior:

- These surfaces use standard Apple navigation motion: smooth, minimal, and hierarchy-preserving
- Card-based and map-based views do not introduce a separate visual-motion language

Design interpretation:

- Motion language stays unified across browse surfaces

Parity implication for Immich:

- Collections, People, and Map should inherit the same transition rules as Library and Viewer
- Motion should not feel different because the route is different

## What makes Photos feel smooth

The smoothness is not one thing. It is several small decisions working together.

### 1. No competing animations

Apple Photos usually lets one structural motion lead:

- open/close
- page
- dismiss
- inspector/edit change

The rest of the UI either remains stable or subtly supports that motion.

### 2. A strong response-to-settle ratio

The first response is quick. The settle is short. The user gets immediate confirmation without waiting around.

### 3. Predictable curves

The app does not jump between very different personalities. It does not use one spring for search, another for paging, another for modal structure, and another for everything else in a way the user notices. It feels unified.

### 4. Spatial honesty

Things appear to come from somewhere and go somewhere.

### 5. Motion paired with loading strategy

Photos also feels smooth because it is not trying to decode giant originals at the exact same moment the user expects interactive transitions. Thumbnail, preview, and original-quality swaps are staged behind the scenes.

Immich already shows signs of this same strategy in the viewer, which is good.

## Current Immich motion inventory

The current implementation already contains a real motion system, but it is distributed across view files.

### 1. Main shell and viewer transition orchestration

Current behavior in [`MainContentView.swift`](/Users/jagravnaik/Projects/immich/desktop-macos/Sources/ImmichMacApp/Views/MainContentView.swift):

- Sidebar selection changes animate with `easeInOut(0.2)` at [MainContentView.swift](/Users/jagravnaik/Projects/immich/desktop-macos/Sources/ImmichMacApp/Views/MainContentView.swift:105)
- Search presentation uses `spring(response: 0.3, dampingFraction: 0.82)` at [MainContentView.swift](/Users/jagravnaik/Projects/immich/desktop-macos/Sources/ImmichMacApp/Views/MainContentView.swift:169)
- Hero open fallback uses `spring(response: 0.35, dampingFraction: 0.88)` at [MainContentView.swift](/Users/jagravnaik/Projects/immich/desktop-macos/Sources/ImmichMacApp/Views/MainContentView.swift:725)
- Hero open uses a two-stage sequence:
  - immediate viewer reveal with `easeOut(0.12)` at [MainContentView.swift](/Users/jagravnaik/Projects/immich/desktop-macos/Sources/ImmichMacApp/Views/MainContentView.swift:741)
  - hero expansion with `easeInOut(0.24)` at [MainContentView.swift](/Users/jagravnaik/Projects/immich/desktop-macos/Sources/ImmichMacApp/Views/MainContentView.swift:747)
  - cleanup after `0.42s` at [MainContentView.swift](/Users/jagravnaik/Projects/immich/desktop-macos/Sources/ImmichMacApp/Views/MainContentView.swift:752)
- Hero close fallback uses `easeOut(0.18)` at [MainContentView.swift](/Users/jagravnaik/Projects/immich/desktop-macos/Sources/ImmichMacApp/Views/MainContentView.swift:770)
- Hero close uses `easeInOut(0.22)` with cleanup after `0.38s` at [MainContentView.swift](/Users/jagravnaik/Projects/immich/desktop-macos/Sources/ImmichMacApp/Views/MainContentView.swift:793) and [MainContentView.swift](/Users/jagravnaik/Projects/immich/desktop-macos/Sources/ImmichMacApp/Views/MainContentView.swift:799)

Assessment:

- Good: the app already thinks in terms of source-aware hero transitions
- Good: the code protects responsiveness by preferring smaller decoded images for hero motion
- Gap: the orchestration relies on several local durations and delayed cleanup points
- Gap: search, sidebar, hero, and edit transitions are not clearly derived from one motion language

### 2. Viewer paging and interactive dismiss

Current behavior in [`PhotoDetailView.swift`](/Users/jagravnaik/Projects/immich/desktop-macos/Sources/ImmichMacApp/Views/PhotoDetailView.swift):

- Live Photo playback fades out with `easeOut(0.4)` at [PhotoDetailView.swift](/Users/jagravnaik/Projects/immich/desktop-macos/Sources/ImmichMacApp/Views/PhotoDetailView.swift:252)
- Page swipe uses `spring(response: 0.28, dampingFraction: 0.9)` with a `280ms` settle window at [PhotoDetailView.swift](/Users/jagravnaik/Projects/immich/desktop-macos/Sources/ImmichMacApp/Views/PhotoDetailView.swift:418) and [PhotoDetailView.swift](/Users/jagravnaik/Projects/immich/desktop-macos/Sources/ImmichMacApp/Views/PhotoDetailView.swift:422)
- Vertical dismiss is gesture-progress-driven through offset, scale, and backdrop opacity at [PhotoDetailView.swift](/Users/jagravnaik/Projects/immich/desktop-macos/Sources/ImmichMacApp/Views/PhotoDetailView.swift:443)
- Horizontal swipe thresholds are currently `100pt` and vertical dismiss threshold is `120pt` at [PhotoDetailView.swift](/Users/jagravnaik/Projects/immich/desktop-macos/Sources/ImmichMacApp/Views/PhotoDetailView.swift:465) and [PhotoDetailView.swift](/Users/jagravnaik/Projects/immich/desktop-macos/Sources/ImmichMacApp/Views/PhotoDetailView.swift:474)
- Pinch-dismiss rebound uses `spring(response: 0.28, dampingFraction: 0.88)` at [PhotoDetailView.swift](/Users/jagravnaik/Projects/immich/desktop-macos/Sources/ImmichMacApp/Views/PhotoDetailView.swift:564)

Assessment:

- Good: this is the strongest motion area in the app
- Good: direct manipulation is already built into the viewer model
- Good: paging and dismiss behavior are conceptually aligned with Photos
- Gap: some polish details still need unification with the hero and toolbar transitions
- Gap: final close behavior depends on coordination outside the viewer, so the whole flow can still feel slightly stitched together

### 3. Library grid and timeline transitions

Current behavior in [`LibraryGridView.swift`](/Users/jagravnaik/Projects/immich/desktop-macos/Sources/ImmichMacApp/Views/LibraryGridView.swift):

- Grid scale changes animate with `easeInOut(0.22)` at [LibraryGridView.swift](/Users/jagravnaik/Projects/immich/desktop-macos/Sources/ImmichMacApp/Views/LibraryGridView.swift:112)
- Timeline view mode changes animate with `easeInOut(0.25)` at [LibraryGridView.swift](/Users/jagravnaik/Projects/immich/desktop-macos/Sources/ImmichMacApp/Views/LibraryGridView.swift:268)
- Keyboard scroll-to-center uses `easeInOut(0.18)` at [LibraryGridView.swift](/Users/jagravnaik/Projects/immich/desktop-macos/Sources/ImmichMacApp/Views/LibraryGridView.swift:290)
- Hover scaling uses `easeOut(0.2)` at [LibraryGridView.swift](/Users/jagravnaik/Projects/immich/desktop-macos/Sources/ImmichMacApp/Views/LibraryGridView.swift:743)
- Selection overlays fade with `easeOut(0.15)` at [LibraryGridView.swift](/Users/jagravnaik/Projects/immich/desktop-macos/Sources/ImmichMacApp/Views/LibraryGridView.swift:760)
- Hover favorite control uses an opacity transition at [LibraryGridView.swift](/Users/jagravnaik/Projects/immich/desktop-macos/Sources/ImmichMacApp/Views/LibraryGridView.swift:790)

Assessment:

- Good: most grid motion is already restrained
- Good: the durations are in the right general range
- Gap: hover scale is slightly more "interactive UI component" than "dense Photos grid"
- Gap: timeline and density changes still need stronger anchor preservation and less sense of generic re-layout

### 4. Search presentation

Current behavior in [`ToolbarSearchField.swift`](/Users/jagravnaik/Projects/immich/desktop-macos/Sources/ImmichMacApp/Views/ToolbarSearchField.swift):

- Search expands and collapses with a trailing-anchored scale plus opacity transition at [ToolbarSearchField.swift](/Users/jagravnaik/Projects/immich/desktop-macos/Sources/ImmichMacApp/Views/ToolbarSearchField.swift:55)
- The whole control uses `spring(response: 0.3, dampingFraction: 0.82)` at [ToolbarSearchField.swift](/Users/jagravnaik/Projects/immich/desktop-macos/Sources/ImmichMacApp/Views/ToolbarSearchField.swift:72)

Assessment:

- Good: the search field is polished
- Gap: this feels like a control animation more than a Photos structural toolbar transition
- Gap: the search field, suggestions surface, and content-state change should read as one browsing-mode shift

### 5. Editing sidebar

Current behavior in [`EditingSidebar.swift`](/Users/jagravnaik/Projects/immich/desktop-macos/Sources/ImmichMacApp/Views/EditingSidebar.swift):

- Auto enhance uses `easeInOut(0.3)` at [EditingSidebar.swift](/Users/jagravnaik/Projects/immich/desktop-macos/Sources/ImmichMacApp/Views/EditingSidebar.swift:47)
- Reset, revert, and done use `easeInOut(0.2)` at [EditingSidebar.swift](/Users/jagravnaik/Projects/immich/desktop-macos/Sources/ImmichMacApp/Views/EditingSidebar.swift:57), [EditingSidebar.swift](/Users/jagravnaik/Projects/immich/desktop-macos/Sources/ImmichMacApp/Views/EditingSidebar.swift:78), and [EditingSidebar.swift](/Users/jagravnaik/Projects/immich/desktop-macos/Sources/ImmichMacApp/Views/EditingSidebar.swift:89)
- Filter selection uses `easeInOut(0.15)` at [EditingSidebar.swift](/Users/jagravnaik/Projects/immich/desktop-macos/Sources/ImmichMacApp/Views/EditingSidebar.swift:162)

Assessment:

- Good: internal tool interactions are light and quick
- Gap: edit mode itself still behaves like "viewer plus a panel"
- Gap: there is no full Photos-style crop-state transition because the crop tool itself is not fully implemented

## Motion parity gaps

These are the main areas preventing the app from feeling like Photos.

### 1. No centralized motion tokens

The app currently hard-codes a family of nearby but different values:

- `0.12`
- `0.15`
- `0.18`
- `0.2`
- `0.22`
- `0.24`
- `0.25`
- `0.28`
- `0.3`
- `0.35`
- `0.4`

This is a normal early-stage pattern, but it is not where a polished desktop app should stop.

Required change:

- Create a shared motion token set for structural, contextual, interactive, and polish-level animation

### 2. Hero orchestration is good, but not yet system-owned

The current hero open and close paths are credible, but they are manually staged and timing-coupled with delayed cleanup.

Required change:

- Move hero transition timing and state progression into a dedicated transition coordinator
- Derive viewer fade, toolbar change, and source-hide state from one transition model

### 3. Edit mode is under-animated in the wrong way

The issue is not "more animation needed." The issue is that edit mode lacks a strong coordinated structural shift.

Required change:

- Treat browse-to-edit as a first-class workspace transition
- Coordinate canvas emphasis, toolbar swap, inspector reveal, and edit-tab state

### 4. Search feels like a widget, not a mode

Photos search feels integrated into the browsing workspace.

Required change:

- Rework toolbar search, suggestion presentation, and search-result state so they animate as a coherent browse transition

### 5. Crop and cleanup are not at parity

Motion parity cannot be achieved where feature parity is missing.

Required change:

- Implement a crop canvas with animated handles, aspect ratio application, and reset behavior
- Add a cleanup mode with its own canvas-state transitions if the product chooses to chase full Photos parity

### 6. Browse-surface transitions are still too route-local

Library, Collections, People, Map, and Albums should feel like one app with one motion language.

Required change:

- Standardize route-level transitions and inspector behavior across all content surfaces

## What is left to implement for true Photos-level parity

This section focuses on the overlap between feature parity and motion parity.

### Must-have before the app can feel like Photos

- Full crop canvas and aspect ratio tools
- Better edit mode entry and exit choreography
- Source-aware open and close across all major browse surfaces, not just the main library
- Consistent viewer-to-grid return behavior
- Unified toolbar-state transitions for browse, view, search, info, and edit
- Pinned and sharing information architecture parity from the broader parity plan
- Better people/group browsing and richer collections surfaces so motion has equivalent destinations to move through
- Watched-folder and import workflows that behave like native desktop flows, not just upload actions

### Important polish that will materially affect feel

- Reduced hover flourish in dense grids
- Stronger anchor preservation during grid density and timeline changes
- More coordinated inspector reveal and dismissal
- Shared transition tokens for overlays, sheets, banners, and side panels
- Better alignment between image loading stages and user-visible transitions

## Recommended implementation architecture

### 1. Add a shared motion module

Create something like:

- `MotionTokens`
- `MotionContext`
- `MotionCoordinator`

Possible responsibility split:

- `MotionTokens`
  - canonical durations
  - canonical curves
  - thresholds for paging/dismiss interactions
- `MotionContext`
  - structural transition category such as browse, focus, edit, search, inspector
- `MotionCoordinator`
  - owns multi-surface transitions such as hero open/close and edit-mode entry

### 2. Define motion tiers

Recommended tiers:

- `structural`
  - route changes
  - viewer open/close
  - edit entry/exit
- `contextual`
  - toolbar search
  - inspector reveal
  - content overlays
- `interactive`
  - page swipe
  - drag dismiss
  - pinch dismiss
- `polish`
  - hover
  - selection fade
  - button feedback

The point is not abstraction for its own sake. The point is consistency.

### 3. Turn hero transition into a formal state machine

Likely states:

- `idle`
- `opening(sourceFrame, seedImage)`
- `focused`
- `interactiveDismissing(progress)`
- `closing(destinationFrame, seedImage)`

Benefits:

- fewer cleanup races
- less manual delayed state clearing
- better sync between viewer visibility and hero overlay visibility
- easier reuse outside the library grid

### 4. Make edit mode structural

Edit entry should coordinate:

- toolbar role swap
- inspector transition
- canvas affordance change
- edit-tab content change

This should be one system transition, not several local animations.

### 5. Treat loading as part of motion design

Keep and deepen the existing thumbnail-to-preview-to-original strategy in the viewer.

Additional goals:

- open transitions should never wait on original image decode
- page transitions should prioritize interactive-size assets
- edit-mode entry should prepare the best available image before exposing heavy tool affordances

## A practical motion-token recommendation

Exact numbers should be tuned in-app, but the system should probably converge toward a smaller set than it has today.

Suggested starting palette:

- `fastFade`
  - around `0.12` to `0.16`
  - hover, overlay fade, control emphasis
- `structuralShort`
  - around `0.18` to `0.22`
  - toolbar swaps, inspector reveal, small context changes
- `structuralMedium`
  - around `0.22` to `0.28`
  - hero open/close, timeline mode change, search presentation if still animated structurally
- `interactiveSpring`
  - one canonical spring family for paging, pinch rebound, and maybe search if search remains spring-based

The important part is not these exact numbers. The important part is that similar events stop inventing their own timings.

## Proposed parity roadmap

### Phase 1. Motion-system cleanup

Build first:

- shared motion tokens
- hero transition coordinator/state machine
- unified viewer open/close timing
- route and inspector transition cleanup

Why first:

- The app already has motion worth refining
- This work improves perceived quality across every existing feature

### Phase 2. Edit-mode parity

Build next:

- proper edit entry/exit transition
- stronger toolbar and inspector choreography
- full crop canvas
- tab-transition standardization

Why second:

- Apple Photos derives a lot of its product confidence from the editor

### Phase 3. Browse-surface consistency

Build next:

- unify motion across Library, Collections, Albums, People, Map, and Memories
- make source-aware open/close work consistently from all major browse surfaces

Why third:

- This is what turns isolated good screens into one coherent desktop app

### Phase 4. Advanced parity surfaces

Build next:

- sharing flows
- pinned destinations
- people groups
- cleanup mode
- import/watch-folder desktop workflows

Why fourth:

- Full Photos parity depends on both feature coverage and the motion system that binds those features together

## Final recommendation

Do not treat animation as a final pass.

If the target is complete parity with Apple Photos, motion has to be treated as a core part of product architecture. The app already has the right instincts:

- source-aware focus transitions
- interactive viewer gestures
- fast grid and paging responses
- staged image loading

What it needs now is consolidation and discipline.

The right next step is:

1. Keep the broader feature-parity plan in [`macos-photos-parity.md`](/Users/jagravnaik/Projects/immich/docs/docs/developer/macos-photos-parity.md)
2. Use this document as the motion companion spec
3. Start implementation by centralizing motion tokens and rebuilding the viewer and edit transitions around one coordinated motion system

## Appendix: current test status

`swift test` in `desktop-macos/` currently fails before test execution because `MockImmichAPIClient` in `desktop-macos/Tests/ImmichMacAppTests/AppStateTests.swift` does not implement the newer `resumeSession(server:accessToken:)` requirement from `ImmichAPIClient`.
