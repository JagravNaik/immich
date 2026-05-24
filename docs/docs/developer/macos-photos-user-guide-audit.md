# Apple Photos User Guide Audit for Immich macOS Parity

This document is a companion to:

- [`macos-photos-parity.md`](/Users/jagravnaik/Projects/immich/docs/docs/developer/macos-photos-parity.md)
- [`macos-photos-motion-parity.md`](/Users/jagravnaik/Projects/immich/docs/docs/developer/macos-photos-motion-parity.md)

It is based on the official Apple Photos user guide for macOS:

- [Photos User Guide for Mac](https://support.apple.com/en-am/guide/photos/welcome/mac)

## Method

This audit combines:

- Direct browser navigation through the live guide and its table of contents
- Extraction of the full table of contents from the guide page
- Review of each linked guide page in the TOC

The current guide exposes 85 entries. This document groups them into product capabilities so Apple’s documented feature surface can be used as a parity checklist for `desktop-macos/`.

## Why this matters

Hands-on app exploration is good for learning the interaction model and motion language.

The user guide is better for completeness.

It surfaces documented product areas that are easy to miss in a quick exploratory session, especially:

- `Recently Saved`
- utility collections like receipts, documents, handwriting, and recently edited
- duplicate detection
- captions and keyword workflows
- Smart Albums
- group photos of people and pets
- memory suppression controls
- third-party editing extensions
- Cinematic video editing
- shared album subscriber management
- multi-library management, repair, and referenced-file workflows
- keyboard shortcuts and gestures

## High-level Apple Photos scope from the guide

Apple Photos is documented as all of the following at once:

- Import and sync client
- Chronological and semantic library browser
- Search and machine-intelligence surface
- Viewer and metadata inspector
- Photo and video editor
- Album and memory organizer
- Shared collaboration system
- Export, slideshow, printing, and project surface
- Multi-library desktop management tool

That reinforces the core parity framing: a true clone cannot stop at "nice grid plus viewer."

## TOC coverage by section

## Top level

- `Get started`
  Apple frames Photos as a browse, organize, find, perfect, and share app, with iCloud Photos positioned as a core part of the baseline setup.

## Import photos and videos

- `Overview of importing`
  Import is a first-class subsystem, not a single file picker. Apple documents multiple ingest paths into one library.
- `Use iCloud Photos`
  Sync is described as whole-library continuity across Mac, iPhone, iPad, Apple TV, Vision Pro, PC, and web.
- `Import from a camera or phone`
  Device import includes practical constraints like Hidden album behavior and import-specific flows.
- `Import from storage devices`
  External storage ingest is explicitly supported.
- `Import from Mail, Safari, and other apps`
  Photos accepts inbound media from other apps, not just devices and folders.
- `Import from another photo library`
  Multiple libraries are assumed and supported.
- `Where are the items I imported?`
  Apple explicitly teaches the library container model, System Photo Library behavior, and where imported media lives.

Parity implication:

- Immich macOS needs a richer ingest model than ad hoc uploads.
- Watched folders, multiple import surfaces, and library-location semantics matter for parity.

## View and find photos

- `Browse your photo library`
  The whole library remains a primary chronological browse surface.
- `Browse photo collections`
  Collections are a top-level discovery surface including Memories, Pinned, Albums, and People & Pets.
- `View photos and videos`
  Single-item viewing is documented as a broad action hub: enjoy, zoom, favorite, edit, share.
- `View photo bursts`
  Bursts are treated as a specific supported media workflow.
- `See photo and video information`
  Metadata includes capture details, recognized people and pets, and detected subjects through Visual Look Up, with event-level contextual knowledge like concerts and sports.

Parity implication:

- The viewer and inspector need to be richer than EXIF display.
- Burst handling is part of the desktop product surface.

## Find photos and videos

- `Find photos and videos by date`
  Chronological browsing includes specific date-based lookup and `Recent Days`.
- `Find and name people and pets`
  People & Pets is a major browse surface, with naming and identity propagation across the library.
- `Find group photos and videos`
  Apple groups people and pets who often appear together into social clusters.
- `Find photos and videos by location`
  Map and place browsing are core flows, not novelties.
- `View photos and videos you recently saved`
  `Recently Saved` is a dedicated semantic collection for items saved from apps and AirDrop.
- `Find your travel photos and videos`
  Trips are location-derived collections.
- `Find receipts, documents, recently edited photos, and more`
  `Utilities` includes behavior- and content-derived collections like recently edited, recently viewed, recently shared, documents, receipts, handwriting, and illustrations.
- `Find screenshots, Live Photos, and more by media type`
  Media-type browsing includes more than favorites and videos; it extends to portraits, time-lapse, and other specialized media categories.

Parity implication:

- Immich parity needs more semantic browse destinations than the current visible basics.
- `Recently Saved`, trips, richer utilities, and people groups are clearly part of Apple’s documented IA.

## Filter and search for photos

- `Filter your photo library`
  Filtering is available inside the library and collections, including edited items and keyword-based views.
- `Search for photos and videos`
  Search covers titles, captions, keywords, dates, and text inside photos.
- `Use Live Text`
  Text in images is directly interactive, not just searchable.
- `Use Visual Look Up`
  Landmark, art, plant, flower, pet, and object understanding is exposed to the user.
- `Isolate and share a photo’s subject`
  Subject lifting is part of the Photos experience, not only a system-level OS trick.
- `Delete photos and videos or recover deleted ones`
  Recoverable deletion is explicitly documented as a core flow.
- `Remove duplicates`
  Duplicate detection is surfaced as a collection in `Utilities`.
- `Hide photos and videos from view`
  Hiding is a supported organization/privacy behavior.
- `Add titles, captions, and more`
  Metadata editing includes titles, captions, favorites, and editable date/time/location.
- `Add keywords`
  Keywords remain a distinct manual classification system.

Parity implication:

- Search parity is broader than text search plus filters.
- Metadata editing, duplicate handling, hidden-item UX, and manual keywords are all part of the Apple baseline.

## Organize photos in albums

- `Create and work with albums`
  Albums are user-authored organization objects and can hold items already present elsewhere.
- `Group albums in folders`
  Album folders are a first-class organizational feature.
- `Create Smart Albums`
  Mac-only Smart Albums automatically gather items based on criteria.

Parity implication:

- Full desktop parity includes album folders and Smart Albums.
- This is a meaningful place where the Mac app should exceed a simpler mobile client.

## View memories

- `Watch memories`
  Memories are movie-like, music-backed collections and can also be user-created.
- `Personalize memories`
  Users can change songs, titles, duration, and included photos.
- `Feature certain people and content less`
  The user can explicitly downrank people, places, days, and holidays in memories and featured photos.

Parity implication:

- Memory support is not just "show a card."
- Personalization and suppression controls are part of the system.

## Edit photos and videos

- `Editing basics`
  Apple presents the editor as sophisticated, nondestructive, and broad across both photo and video.
- `Crop and straighten photos and videos`
  Crop includes preset/custom aspect ratios, original ratio constraints, and straighten support.

Parity implication:

- Immich’s crop experience needs a real on-canvas tool and ratio workflow, not a placeholder.

## Adjust the look of a photo

- `Add filters`
  Filtering remains a supported editing path, but Apple now explicitly points newer iPhone captures toward Photographic Styles.
- `Adjust light, exposure, and color`
  Apple supports Auto Enhance plus expanded fine-grained controls, and even documents the `A` shortcut for adjustments.
- `Remove distractions and imperfections`
  Apple distinguishes `Clean Up` and `Retouch`, with Apple Intelligence called out for background-object removal.
- `Remove red-eye`
  Red-eye remains a distinct tool and is compatible with Live Photos.
- `Adjust white balance`
  White balance is a standalone adjustment with copy/paste edits support.
- `Apply curves adjustments`
  Curves includes per-channel control and histogram-driven editing.
- `Apply levels adjustments`
  Levels is also a dedicated tool with black point, shadows, midtones, highlights, and white point.
- `Adjust definition`
  Definition is documented as contour, shape, and local-contrast control.
- `Adjust specific colors`
  Selective Color supports hue, saturation, and luminance for up to six colors.
- `Reduce noise`
  Noise reduction supports low-light cleanup and RAW workflows.
- `Sharpen a photo`
  Sharpen is its own tool.
- `Apply a vignette`
  Vignette includes darkness, size, and softness tuning.
- `Change Photographic Styles`
  Apple documents a style system specific to newer iPhone captures and Apple silicon Macs.
- `Change a Portrait mode photo`
  Portrait editing includes depth-of-field and six studio lighting effects.
- `Write or draw on a photo`
  Markup is an integrated workflow for drawing, shapes, stickers, crop, and rotate.
- `Use other apps when editing in Photos`
  Photos supports third-party editing extensions.
- `Change a Live Photo`
  Live Photos support edit workflows with some tool-specific limitations.
- `Change a video`
  Videos support much of the same adjustment language as photos, including HDR workflows.
- `Edit a Cinematic mode video`
  Cinematic mode video includes focus-point editing and depth-of-field control.

Parity implication:

- The current Immich editor is far from Apple’s documented depth.
- Full parity means not only more sliders, but more editing modes, richer media-specific tools, and extension-aware architecture.

## Share photos and videos

- `Share photos and videos`
  General sharing spans Mail, Messages, installed share targets, and AirDrop.

Parity implication:

- Native macOS sharing should stay deeply integrated, not treated as an afterthought.

## Shared albums

- `Turn on Shared Albums`
  Shared Albums support likes, comments, subscriber posting, and even public websites for non-iCloud users.
- `Create or join shared albums`
  Shared album membership and collaboration are part of the normal workflow.
- `Add, remove, and edit photos and videos in a shared album`
  Shared albums are editable collaborative spaces.
- `Add and remove people in a shared album`
  Subscriber management, notifications, posting permissions, and public-link sharing are all surfaced in-product.

Parity implication:

- Apple’s `Sharing` area is a substantial feature cluster, not a single share sheet.

## iCloud Shared Photo Library

- `What is iCloud Shared Photo Library?`
  Shared Library is a separate collaborative library for up to six participants total.
- `Set up or join a shared library`
  Joining or setting up a shared library is a major system workflow.
- `View a shared library`
  Users can switch between personal, shared, or combined views.
- `Add photos to a shared library`
  Photos supports manual moves plus suggestions, with camera-based auto-routing on iPhone/iPad.
- `Remove photos from a shared library`
  Contribution ownership matters when moving items back out.
- `Export photos, videos, slideshows, and memories`
  Export includes format, naming, folder structure, and Live Photo still export options.
- `Export a Live Photo as an animated GIF`
  GIF export is explicitly supported.
- `Export a still photo from a video`
  Frame export from video is documented.

Parity implication:

- A real Photos clone with Immich backend likely needs a deliberate replacement story for both `Shared Albums` and `Shared Library`, even if the underlying sync model differs from iCloud.

## Create slideshows and projects

- `Create slideshows`
  Apple supports both quick instant slideshows and more deliberate slideshow creation.
- `Create projects using third-party apps`
  Photos can hand off into third-party project apps for books, cards, calendars, and more.

Parity implication:

- Slideshows and project handoff are part of the desktop app’s long-tail feature surface.

## Print your photos

- `Print your own photos`
  Photos supports standard sizes, custom sizes, contact sheets, and printer-aware black-and-white or color output.
- `Order professional prints`
  Apple positions print-ordering through third-party apps like Mimeo Photos.

Parity implication:

- Printing is still part of the documented Mac experience, and desktop parity should acknowledge it.

## Manage your photo library

- `Photo library overview`
  Apple teaches the System Photo Library concept and the fact that only that library participates fully in iCloud-based sharing and sync.
- `Create additional libraries`
  Multiple libraries are a standard supported workflow.
- `Back up your library`
  Apple explicitly documents local backup expectations even when iCloud Photos is enabled.
- `Restore from Time Machine`
  Restore is a user-facing workflow, not only an admin concern.
- `Repair your photo library`
  Repair is a supported recovery tool.
- `Change where photos and videos are stored`
  Referenced files outside the library are a documented feature.
- `Photos settings`
  Settings cover library location, privacy, autoplay, HDR, memories, importing, location sharing, enhanced visual search, iCloud, and Shared Library behaviors.
- `Keyboard shortcuts and gestures`
  Apple documents keyboard and pointer workflows as part of the product surface.
- `Copyright and trademarks`
  Not product-relevant for parity.

Parity implication:

- The desktop app needs stronger library-management semantics, settings coverage, and power-user workflows.
- Referenced-file behavior is especially relevant because it overlaps with an Immich-style desktop ingest model.

## Additional parity gaps surfaced by the guide

Compared with the current `desktop-macos/` implementation, the guide reinforces several gaps or partially implemented areas:

- `Recently Saved` as a dedicated collection
- richer `Utilities` coverage including duplicates, documents, receipts, handwriting, illustrations, recently edited, recently viewed, and recently shared
- trips and travel collections
- people-and-pets group collections
- burst browsing
- title, caption, date, time, and location editing in a user-facing metadata workflow
- manual keywords
- album folders
- Smart Albums
- memory personalization and "show less" controls
- proper crop ratio workflow
- `Clean Up` parity
- deeper adjustment stack parity
- Photographic Styles parity
- Portrait lighting and depth editing parity
- Markup workflow
- third-party editing extensions
- fuller video editing parity, including Cinematic
- shared album management area
- shared-library-style collaboration model or a deliberate Immich replacement
- export depth beyond simple save/share
- slideshows
- printing/project integration
- multiple library workflows
- referenced-file workflows
- recovery and repair UX
- broader settings parity
- stronger keyboard and gesture coverage

## Product-level conclusion

The official guide confirms that Apple Photos on macOS is a much broader desktop product than a media browser with editing.

If the target is "complete parity with macOS Photos, but with Immich instead of iCloud," the app must be designed as a native photo workstation with:

- multiple ingest paths
- a large semantic browse model
- a serious editor for both photos and video
- collaborative sharing models
- export, slideshow, and printing workflows
- library-management and recovery tooling
- strong keyboard and desktop-native behavior

The current Immich app is already a strong start, but the guide makes it clear how much of Apple Photos lives outside the most obvious library-viewer-editor loop.
