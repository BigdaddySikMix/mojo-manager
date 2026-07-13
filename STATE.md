A1 MOJO MANAGER — MASTER STATE FILE
Claude Code Handoff Document
Date: July 13, 2026 — updated end of session
Owner: JD Ploetz, Area Manager — ABQ / TUS / SFE / SRV
Built with: Claude (claude.ai) — transitioning to Claude Code

═══════════════════════════════════════════════════════
READ THIS FIRST — HOW THIS PROJECT WORKS
═══════════════════════════════════════════════════════
This is a single HTML file application. Everything — HTML, CSS, and
JavaScript — lives in one file: index.html. There is no build system,
no npm, no framework, no dependencies to install. It runs directly in
Chrome via GitHub Pages. Do not suggest splitting into multiple files
or adding a build pipeline. The single-file architecture is intentional
and must be preserved.

The developer (JD) knows enough to be dangerous with code. He learns
by doing with guided steps. He communicates casually and often via
voice-to-text. He moves fast and tests iteratively. Keep responses
concise and action-oriented.

EVERY build gets a version bump on the third digit. No exceptions.
Version string must be updated in ALL FOUR locations simultaneously:
  1. <title> tag
  2. #ver-lbl corner badge
  3. #startup-sub splash screen
  4. #sl status bar

═══════════════════════════════════════════════════════
LIVE APP & REPOSITORY
═══════════════════════════════════════════════════════
Live URL:   https://bigdaddysikmix.github.io/mojo-manager/
GitHub:     github.com/BigdaddySikMix/mojo-manager
Main file:  index.html (the entire app)
Current version: v2.1.17

Deployment process:
  1. Edit index.html
  2. Push to GitHub (main branch)
  3. Wait for green checkmark in GitHub Actions
  4. Hard reload Chrome (CTRL+SHIFT+R)
  5. Confirm version badge in corner

Custom domain: a1mojo.com (Namecheap, registered through May 2027)
Domain status: DECOUPLED — corporate SSL inspection blocks it on work PC
IT contact: Brent — needs domain whitelist + SSL inspection bypass
Fallback: github.io URL works fine in the meantime

═══════════════════════════════════════════════════════
GITHUB REPO TRANSFER — STATUS & NOTES
═══════════════════════════════════════════════════════
Goal: Move repo from BigdaddySikMix (personal) to JploetzA1 (A1 org)

Current status: PENDING — transfer was initiated to @JploetzA1 but
acceptance email was never received/accepted. Transfer is sitting in
limbo. To restart: go to BigdaddySikMix/mojo-manager → Settings →
Danger Zone → Abort Transfer, then re-initiate.

The problem: JploetzA1 is an ORGANIZATION account (not personal).
The email jploetz@a1garage.com is on the account but corporate
email filtering (Sophos) wraps all GitHub links, making acceptance
links potentially broken. GitHub emails DO arrive at that inbox
(confirmed) but the acceptance link may be mangled by Sophos.

Fix before next attempt:
  1. Log into JploetzA1 org account
  2. Go to github.com/settings/emails
  3. Add a personal email (Gmail etc) as backup
  4. Abort current pending transfer on BigdaddySikMix
  5. Re-initiate transfer — acceptance email goes to personal email
  6. Click accept link from personal email

After transfer completes:
  - Live URL changes to: jploetzA1.github.io/mojo-manager
  - Update any hardcoded URLs in A1_MOJO_INTRO.html
  - Reinstall PWA from new URL
  - a1mojo.com custom domain: update in new repo GitHub Pages settings
    (DNS records on Namecheap don't need to change)

NOTE: GitHub sets up automatic redirects from old URL — nothing
breaks immediately but don't rely on redirects long term.

═══════════════════════════════════════════════════════
CLAUDE CODE ACCESS NOTE (added 7/13/26)
═══════════════════════════════════════════════════════
Enterprise Claude account (work) CANNOT see personal GitHub
repos (BigdaddySikMix) — confirmed via testing, likely an
enterprise policy restriction, not a GitHub-side issue.

WORKAROUND: Use personal Claude.ai account (Pro) to access
Claude Code for this repo. Personal account connects to
BigdaddySikMix/mojo-manager without issue.

Once the GitHub org transfer to JploetzA1 completes, repo
will live under the org account and should be accessible
from enterprise Claude natively — this workaround becomes
unnecessary at that point.

═══════════════════════════════════════════════════════
TECH STACK — COMPLETE
═══════════════════════════════════════════════════════
- Single HTML file — HTML + CSS + JavaScript, no framework
- File System Access API — folder picker, file reading, write access
- IndexedDB — persists folder handles across sessions (2 handles:
    "folder" = main audio folder
    "rotator_folder" = rotator clip subfolder)
- localStorage — persists all user settings:
    a1mojo2_carts     — slot assignments, volumes, points
    a1mojo2_time      — call time setting
    a1mojo2_meetlink  — Teams meeting URL
    a1mojo2_vidurl    — saved video URL
- HTML5 Audio — all playback (no Web Audio for playback)
- Web Audio API — ONLY used for waveform decoding in trim/cue editor
- Google Fonts — Bebas Neue, Share Tech Mono, Rajdhani
- GitHub Pages — hosting

NO external JS libraries. NO npm. NO build tools.

═══════════════════════════════════════════════════════
APPLICATION ARCHITECTURE
═══════════════════════════════════════════════════════

CONSTANTS:
  MUSIC_SLOTS = 5       (top music channels)
  SFX_PER_PAGE = 12     (SFX grid slots per page)
  SFX_PAGES = 3         (number of SFX pages)
  SFX_TOTAL = 36        (SFX_PER_PAGE * SFX_PAGES)
  TOTAL = 41            (MUSIC_SLOTS + SFX_TOTAL)
  FADE_RESET_VOL = 0.9  (volume after auto-fade)

CART OBJECT STRUCTURE (one per slot, array of 41):
  {
    idx,          // slot index 0-40
    name,         // filename without extension (null if empty)
    file,         // blob URL for playback
    fileHandle,   // File System Access API handle
    audio,        // current Audio() object (null if stopped)
    playing,      // boolean
    duration,     // total seconds
    startTime,    // Date.now() when play started (for countdown)
    volume,       // 0.0 - 1.0
    fadeSpeed,    // ms for auto-fade (2000/5000/10000)
    fadeInterval, // interval ID during fade
    auto,         // Play Next toggle (music channels only)
    _queuePos,    // saved seek position when stopped
    _seeking,     // true while user drags seek bar
    _cleared,     // tombstone — slot was intentionally cleared
    inPoint,      // seconds — playback start (0 = beginning)
    outPoint,     // seconds — playback stop (0 = play to end)
    cuePoint      // seconds — fire next seq slot here (0 = disabled)
  }

SLOT INDEX MAPPING:
  0-4    = Music channels 1-5
  5-16   = SFX Page 1 (slots 1-12)
           NOTE: index 5 (SFX slot 1) is the ROTATOR SLOT
  17-28  = SFX Page 2
  29-40  = SFX Page 3

MUSIC CHANNEL INDEX: idx < MUSIC_SLOTS (idx < 5)
SFX INDEX: idx >= MUSIC_SLOTS (idx >= 5)
ROTATOR: always idx === MUSIC_SLOTS (idx === 5), managed separately

═══════════════════════════════════════════════════════
PERSISTENCE LOGIC — CRITICAL
═══════════════════════════════════════════════════════
Three possible states per saved cart slot:
  {name, auto, volume, fadeSpeed, inPoint, outPoint, cuePoint}
    → slot has an assigned file
  {cleared: true}
    → slot was intentionally cleared, NEVER backfill this slot
  null
    → slot was never assigned, ok to backfill from folder

On loadFromFolder():
  1. Read saved cart data from localStorage
  2. For each saved slot with a name, find matching file in folder
  3. RESTORE inPoint, outPoint, cuePoint from saved data (critical —
     easy to forget this, was a major bug in v2.1.9/v2.1.10)
  4. Backfill only null slots with unassigned files
  5. Never touch _cleared slots

═══════════════════════════════════════════════════════
FEATURE SET — v2.1.17 COMPLETE
═══════════════════════════════════════════════════════

HEADER:
  - A1 logo (A1-logo.png on GitHub Pages)
  - App title, area manager console subtitle
  - Live clock (updates every second)
  - Date display
  - Version badge (#ver-lbl)
  - Branding: Dabbing Pug Productions / A1 Garage Door Service ONLY

FOLDER BAR:
  - SELECT AUDIO FOLDER — yellow pulsing animation until picked
  - Folder name + file count display
  - RELOAD button — rescans folder, restores all saved points
  - Folder handle saved to IndexedDB key "folder"
  - Auto-restores on every open via requestPermission()

MOJO STRIP:
  - TIME TILL MOJO countdown (HH:MM → MM:SS → GO! → pulse red)
  - Editable call time + AM/PM indicator
  - JOIN MOJO CALL button → opens Teams link in new tab
  - Teams link field (localStorage)
  - OPEN URL button + field → opens any URL (localStorage)

MASTER DECK:
  - Single slider ducks all 5 music channels simultaneously
  - Does NOT affect SFX slots
  - RESET button returns to 100%
  - MASTER FADE button (red/orange, emergency accent) — fades out and
    stops EVERY currently playing slot at once: all music channels,
    all playing SFX slots, and the rotator if it's playing
  - Uses the same 2-second linear fade curve as the per-slot fade (↓)
    buttons, regardless of each music channel's own SPD setting
  - Does NOT chain Play-Next (PN) to the next track — full stop, no
    auto-advance after a master fade
  - Does NOT clear slot assignments or tombstones — just stops playback,
    same as a normal STOP/fade-out
  - Guarded by masterFading flag so rapid re-clicks don't double-fire
  - State: masterFading; functions: masterFadeAll(), fadeAndStopSlot(),
    fadeAndStopRotator()

MUSIC CHANNELS (5 slots, idx 0-4):
  - Drag handle (⠿) for reordering via drag/drop
  - Channel number, track name
  - Countdown timer (pre-roll duration at rest, live countdown playing)
  - CLR button (tombstone — cleared slot persists across reopen)
  - PLAY / STOP buttons
  - Seek bar with scrub and position display
  - Volume fader with color gradient and percentage
  - Speed dots: Fast(2s) / Med(5s) / Slow(10s) for fade speed
  - Auto-fade button (↓) — fades to 0 then resets volume to 90%
  - Play Next (PN) toggle — chains to next channel on ended
  - Drag & drop from OS file system or between slots
  - Right-click LOADED slot → choice menu (Editor / Change File)
  - Right-click EMPTY slot → file picker list from folder

SFX GRID (36 slots, 3 pages of 12):
  - SLOT 1 PAGE 1 (idx 5) = ROTATOR SLOT (special, see below)
  - Pre-roll duration shown at rest (bottom right)
  - Live countdown while playing (green, bottom right)
  - CLR button (top right) — tombstone persistence
  - Slot number (bottom left, drag handle)
  - Click to play/stop
  - Fade button (↓, bottom left of loaded slots) — 1.5s fade out
  - ✂ indicator = trim points set
  - ◆ indicator = cue point set
  - Right-click LOADED slot → choice menu (Editor / Change File)
  - Right-click EMPTY slot → file picker list from folder

PAGE TABS (PAGE 1 / PAGE 2 / PAGE 3):
  - Click to switch pages
  - Hover over inactive page tab → floating preview list appears
    showing slot number + clip name for all loaded slots on that page
  - Clicking an item in the preview jumps to that page
  - Preview appears above/left of tab (not below/right — goes offscreen)

ROTATOR SLOT (SFX Page 1, Slot 1, idx 5):
  - Black/red dashed animated border — visually distinct
  - Manages its own separate subfolder (not the main audio folder)
  - Folder handle saved to IndexedDB key "rotator_folder"
  - Each click plays next clip in rotation, auto-advances index
  - Shows current clip name
  - Right-click menu:
      🎯 Pick Specific Clip — scrollable list, click to jump to clip
      📁 Set Rotator Folder — opens folder picker
      ↻  Shuffle Order — randomizes rotation
      ⏮  Reset to First — goes back to index 0
  - Standalone only — not integrated with sequencer
  - State: rotatorDirHandle, rotatorFiles[], rotatorPos, rotatorAudio,
           rotatorPlaying

TRIM / CUE EDITOR (modal):
  - Opens via right-click → "Open Trim/Cue Editor" on any loaded slot
  - Also opens via right-click on a filled sequencer slot (CUE MODE)
  - Real waveform rendered via Web Audio API decodeAudioData()
  - Decoded buffer cached in editorDecodedBuffer
  - Mouse wheel zoom on timeline (up to 20x, zooms around cursor)
  - Zoom-aware: markers, playhead, ticks all respect zoom
  - Draggable markers: IN (green), CUE (amber), OUT (red)
  - PLAY / STOP transport
  - SET IN / SET CUE / SET OUT — stamps at current playback position
  - CLEAR ALL — resets all points
  - SAVE — writes to cart object + localStorage, re-renders slot
  - Points follow clip everywhere (sequencer, direct play)

  Editor state variables:
    editorIdx, editorCueMode, editorAudio, editorPlaying
    editorDuration, editorInPct, editorOutPct, editorCuePct
    editorHasCue, editorAnimFrame, editorDragging
    editorZoom, editorZoomOffset, editorDecodedBuffer

SEQUENCER (4 slots at bottom):
  - Drag ANY cart (music OR SFX) into seq slots
  - Right-click filled seq slot → opens editor in CUE MODE
  - FIRE SEQ → plays slots in order
  - CUE POINT: next clip fires IMMEDIATELY, current fades underneath
  - OUT POINT: stops clip, chains next after 250ms
  - IN POINT: uses canplay event (not loadedmetadata — avoids stutter)
  - advanced guard prevents double-firing on ended event
  - STOP SEQ / CLEAR buttons
  - State: seqSlots[4], seqActive, seqQueue[]

RIGHT-CLICK SYSTEM:
  - Loaded slot → showSlotChoiceMenu() → 2 options:
      ✂ Open Trim/Cue Editor → openEditor(idx, false)
      📁 Change File → openFileListForSlot() with 50ms delay
        (delay is CRITICAL — do not remove the setTimeout)
  - Empty slot → showCtxMenu() directly
  - Rotator → showRotatorCtx()
  - Seq slot → openEditor(idx, true) directly

═══════════════════════════════════════════════════════
KNOWN BUGS & TODO — NEXT BUILD (v2.1.18)
═══════════════════════════════════════════════════════

1. EDITOR CURSOR CLICK-TO-SEEK (HIGH PRIORITY)
   Clicking timeline should place playhead AND set PLAY start position.
   Currently PLAY always starts from inPoint.
   Fix: store editorPlayStartPct, update on click, use in editorPlay().

2. SEQUENCER BOUNCE TO FILE (HIGH PRIORITY)
   Render entire sequence with crossfades into single WAV download.
   Approach: OfflineAudioContext, render each clip at correct position
   with fade automation, export via bufferToWav().

3. ROTATOR SLOT — PRE-ROLL DURATION + LIVE COUNTDOWN
   Show clip duration at rest, live green countdown while playing.
   Same behavior as SFX slots.

4. ROTATOR FILE COUNT INDICATOR
   Show "X files" on rotator slot so JD knows how many drops loaded.

═══════════════════════════════════════════════════════
CRITICAL BUGS FIXED — DO NOT RE-INTRODUCE
═══════════════════════════════════════════════════════

1. loadFromFolder() must restore inPoint/outPoint/cuePoint from
   saved data. Easy to miss when refactoring. Fixed v2.1.10.

2. playSeqStep() needs "advanced" guard flag — prevents double-firing
   when cue watcher AND "ended" event both fire. Fixed v2.1.12.

3. inPoint seek stutter: use "canplay" not "loadedmetadata".
   loadedmetadata fires before browser is ready to seek. Fixed v2.1.12.

4. Right-click "Change File": dedicated openFileListForSlot() with
   50ms setTimeout delay. DO NOT remove that delay. Fixed v2.1.16.

5. Sequencer cue crossfade: fire next clip IMMEDIATELY at cue point,
   fade current underneath simultaneously. Fixed v2.1.13.

6. Ghost cue point: editorHasCue set from cart.cuePoint > 0 BEFORE
   setting editorCuePct. Order matters. Fixed v2.1.11.

7. Context menus close immediately: all right-click menus use
   setTimeout 50ms before adding document click listener. Fixed throughout.

═══════════════════════════════════════════════════════
RELATED STANDALONE TOOL
═══════════════════════════════════════════════════════
AudioTrimEditor_v1.0.html — standalone audio trim/cue tool
  - No Mojo Manager dependency
  - Multi-file drag and drop, file list sidebar
  - Real WAV export with IN/OUT trim
  - Cue point baked into exported filename (_cue_MM_SS)
  - Used by JD's audio clip extraction project
    (YouTube → audio → AI quote extraction → trim → drops)

═══════════════════════════════════════════════════════
HARDWARE & SETUP CONTEXT
═══════════════════════════════════════════════════════
- Office: Poly tabletop speaker/mic
- Audio passes naturally into Teams — NO screen share needed
- Work PC has corporate SSL inspection (Sophos)
- File System Access API: Chrome only (not Firefox/Safari)
- External drive drag-drop: silent fail if disconnected (known bug)

═══════════════════════════════════════════════════════
VERSION HISTORY
═══════════════════════════════════════════════════════
v1.x     Basic soundboard, slot memory, folder persistence,
         mic indicator, YouTube button, TIME TILL MOJO countdown
v2.0.5   Major rebuild: 4 music channels, 36 SFX slots, sequencer,
         drag/drop everywhere, speed dots, faders, fade/PN
v2.1.0   Seek bar on music channels, larger SFX font
v2.1.1   Stop keeps seek position, big countdowns, SFX timers
v2.1.2   Pre-roll duration display, OPEN URL, Master Deck
v2.1.3   CLR buttons on music channels and SFX slots
v2.1.4   Right-click context menu on any slot
v2.1.5   Fixed context menu file loading (fileHandle map fix)
v2.1.6   Tombstone logic — CLR'd slots persist across reopen
v2.1.7   Branding cleanup, 5th music channel, Rotator slot
v2.1.8   Trim/Cue Editor, cue points on sequencer, ✂ ◆ indicators
v2.1.9   Fixed playSeqStep to honor cue points
v2.1.10  Fixed cue/in/out points not restoring after folder reload
v2.1.11  Music channels draggable into sequencer, ghost cue fixed
v2.1.12  Fixed inPoint stutter, added advanced guard
v2.1.13  Fixed cue timing — next clip fires immediately at cue point
v2.1.14  Right-click choice menu, Rotator pick specific clip,
         page tab hover preview, SFX fade button
v2.1.15  Change File fix, page preview up-left, editor wheel zoom
v2.1.16  Page preview clickable, Change File conflict resolved
v2.1.17  MASTER FADE button — emergency fade+stop of all playing
         music/SFX/rotator audio at once, 2s fade curve

═══════════════════════════════════════════════════════
FUTURE WISHLIST
═══════════════════════════════════════════════════════
- Tooltip/hover help system (full board instruction layer)
- Saved video URL bank (6-10 slots with nicknames)
- GitHub repo transfer to JploetzA1 org (see transfer notes above)
- Standalone A1 Mojo Call App (needs real backend dev)
- KGDS — The Garage Door Sound (internet radio for field techs)
- Useless Information Podcast

═══════════════════════════════════════════════════════
WORKFLOW NOTES FOR CLAUDE CODE
═══════════════════════════════════════════════════════
- Always read full index.html before making changes
- Test by opening file directly in Chrome — no server needed
- Audio APIs require user gesture — autoplay won't work on load
- File System Access API: Chrome only
- IndexedDB name: "a1mojo201_db", objectStore: "handles"
- localStorage keys all prefixed "a1mojo2_"
- grep is your friend — everything is in one file
- JD tests each version and reports back before next patch
- Don't batch too many changes at once
- Update state file at end of every session
