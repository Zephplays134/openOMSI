# Safety and Usability Guide for openOMSI

> **Goal:** Make openOMSI safe, intuitive, and accessible for all users while maintaining compatibility and performance.

---

## Table of Contents

1. [Safety](#safety)
2. [Ease of Use](#ease-of-use)
3. [Error Handling & Recovery](#error-handling--recovery)
4. [Accessibility](#accessibility)
5. [Performance & Stability](#performance--stability)
6. [Security](#security)

---

## Safety

### Input Validation & Bounds Checking

- **File Path Handling:**
  - Validate all user-provided paths (OMSI 2 root, mod folders, archives) before access
  - Sanitize paths to prevent directory traversal attacks
  - Provide clear error messages when paths are inaccessible or invalid
  - Example: "Invalid path: /path/to/OMSI 2 — this folder does not contain Omsi.exe"

- **Configuration File Integrity:**
  - Validate `settings.cfg`, `keyboard.cfg`, and `gamectrler.cfg` before parsing
  - Use default values for missing or malformed keys instead of crashing
  - Back up user configs before major version upgrades
  - Warn users if their config is deprecated: "Your settings.cfg uses an old format; we've migrated your settings. [View changes]"

- **Mod & Archive Validation:**
  - Scan archives for oversized files before extracting (warn if > 1 GB)
  - Detect circular dependencies in mods
  - Prevent mods from writing to system folders (enforce content folder only)
  - Check disk space before installation

### Runtime Safety

- **Memory & Resource Limits:**
  - Enforce texture memory budget (`texture_memory` setting) with fallback to lower quality
  - Implement graceful degradation: reduce passenger count, traffic, or draw distance if memory is low
  - Add warnings: "Running low on texture memory; consider reducing draw distance"
  - Monitor GPU memory on supported platforms and adjust accordingly

- **Thread Safety:**
  - Ensure all worker threads have error handlers to prevent silent failures
  - Log unhandled panics in worker threads (scripting, audio, networking)
  - Provide recovery path: "A background task failed; the game will continue with reduced features"

- **Physics & Collision:**
  - Clamp unrealistic vehicle speeds before physics calculations (prevent numerical instability)
  - Validate wheel positions to prevent clipping through terrain
  - Ensure collision detection handles degenerate geometry gracefully

### Data Integrity

- **Save State Protection:**
  - Atomic writes for personality files (`.odr`), save situations, and settings
  - Create backups before writing: `settings.cfg.bak`, `settings.cfg.previous`
  - Validate save file format before loading; auto-repair if corrupted (or prompt user)
  - Hash check for critical data (profile experience points, completed duties)

- **Network Safety (LAN play):**
  - Rate-limit incoming packets to prevent DoS
  - Validate all peer messages before applying state changes
  - Drop invalid variable updates (out-of-range floats, injection attempts in strings)
  - Example: "Player's bus speed exceeds physics limits; update rejected"

---

## Ease of Use

### First-Time Setup

- **Streamlined Onboarding:**
  1. Auto-detect OMSI 2 folder on first launch (search Steam, common paths, user's Documents)
  2. If not found, open a large, friendly file picker with instructions: "Select your OMSI 2 folder (the one with **Omsi.exe**, **maps**, and **Vehicles**)"
  3. Show a progress bar with status: "Validating OMSI 2 installation…"
  4. Display a checklist: ✓ Found Omsi.exe, ✓ Found maps, ✓ Found Vehicles, ✓ Stock content detected
  5. Offer quick-start: "Ready to drive! [Start your first duty]"

- **Launcher Visibility:**
  - Make the launcher window larger and more prominent on first run
  - Highlight the **Drive** page with an arrow pointing to "Start the duty"
  - Show tooltips on hover: "Choose a bus, map, and duty here"

### Error Messages

Replace all technical jargon with user-friendly text:

| Current (Technical) | Improved (User-Friendly) |
|---|---|
| "VkPhysicalDeviceLost" | "The graphics device was lost. Try updating your driver." |
| "Failed to load texture: path/to/texture.tga" | "Texture problem in [Mod Name]. The game will use a fallback. Report this on Discord." |
| "Thread panicked: index out of bounds" | "An internal error occurred. Your game was saved. [Send error report]" |
| "Cannot create window" | "Failed to create game window. Check your graphics settings or restart the game." |

**Implementation:**
- Map Rust `panic!()` messages to user-facing text in `crates/omsi-app/src/error_handler.rs`
- Always offer a next step: "What should I do?" button linking to troubleshooting

### Progressive Disclosure

- **Settings Page:**
  - Hide advanced options by default (Graphics API, debug switches, GPU limits)
  - Show only: Quality, Resolution, Driving Controls, Language
  - Add a "More Options" or "Advanced" toggle to reveal power-user settings
  - Organize by frequency of change (most common first)

- **Controls Page:**
  - Default to a simple preset (WASD or Arrows) with clear instructions
  - Only show full `keyboard.cfg` details if user clicks "Customize all keys"
  - Highlight conflicting keys in red with a suggestion: "Both steering and wiper use W. Change wiper to [  ]?"

### Consistent Terminology

- Use the same term everywhere:
  - "OMSI 2 folder" (not "installation", "root", "game path")
  - "Mods" (not "add-ons", "extensions", "packages")
  - "Duty" (not "run", "tour", "mission")
  - "Depot" (not "garage", "terminal")

### Help & Documentation

- **In-Game Help:**
  - Add "?" icon next to complex settings that opens a tooltip or wiki link
  - Example: **?** next to "Texture memory" → "How much GPU memory to use for images. Increase for better quality, decrease if the game stutters."
  - Links to relevant docs: "docs/USER_GUIDE.md", GitHub Discussions

- **Contextual Error Links:**
  - Every error message has a "Learn more" or "Troubleshoot" link
  - Links point to `docs/` for offline access or GitHub issues for online help

---

## Error Handling & Recovery

### Graceful Degradation

| Failure | Current Behavior | Improved Behavior |
|---|---|---|
| Mod file corrupt | Crash or undefined behavior | Skip mod with warning; suggest disabling it |
| Missing texture | Fallback to magenta placeholder (confusing) | Use gray/white fallback; log which mod has the issue |
| Script error | HUD might freeze or glitch | Execute default behavior; log script error with line number |
| Audio device lost | Silent failure or crash | Disable audio and continue; show HUD notification |

### Recovery Tools

- **Repair on Launch:**
  - `openomsi --verify-installation` checks all mod archives and missing content
  - Offers to remove broken mods or download missing files
  - Takes ~10 seconds for typical setup

- **Reset to Defaults:**
  - Settings → General → "Reset all settings" (with confirmation)
  - Saves current settings to `settings.cfg.backup` first
  - Offers one-click restore: "Your old settings are here: [Restore]"

- **Game Crash Recovery:**
  - On startup, detect if game crashed last time
  - Offer: "Restore your last session?" or "Start fresh?"
  - Show the crash log: "Last error: [details]. [Send to developers]"

---

## Accessibility

### Visual

- **High Contrast Mode:**
  - Add a "High Contrast" toggle in Settings → Display
  - Increase all UI text contrast to WCAG AA standard (at least 4.5:1)
  - Expand HUD icons for easier visibility

- **Text & Font:**
  - Make text size adjustable in the launcher (50–200%)
  - Use clear, sans-serif fonts (Roboto is good; consider dyslexia-friendly options)
  - Support sub-pixel rendering on high-DPI displays

- **Color Blind Support:**
  - Don't rely on color alone to convey information (e.g., traffic light status)
  - Add symbols or text labels alongside colors
  - Provide a color-blind mode that swaps hard-to-distinguish colors

### Audio

- **Captions & Descriptions:**
  - Add optional on-screen captions for in-game radio dialog
  - Provide text descriptions of audio cues (horn, brake, engine)
  - Support text-to-speech for menu navigation

### Motor Control

- **Customizable Controls:**
  - Allow remapping every action to any input (keyboard, gamepad, wheel)
  - Support adaptive controllers (e.g., Xbox Adaptive Controller)
  - Provide a "Toggle mode" for sustained buttons (hold brake, then press again to release)

- **Eye Tracking:**
  - Document support for eye trackers (e.g., Tobii)
  - Test with popular accessibility hardware

---

## Performance & Stability

### Profiling & Diagnosis

- **Built-in Profiler:**
  - Add a **Settings → Debug → "Show performance stats"** toggle (hidden by default)
  - Display: FPS, frame time, GPU/CPU memory, texture count, draw calls
  - Log to file: `~/.openomsi/performance.log` every second
  - Example: `[12:34:56.789] FPS: 60, Frame: 16.7 ms, GPU memory: 2.1 GB, Textures: 1234`

- **Auto-Detection of Performance Issues:**
  - If FPS drops below 30 for > 5 seconds, show a suggestion: "Frame rate is low. Try: [Reduce draw distance] [Lower texture quality] [More info]"
  - Log the state at that moment for later analysis

### Stability Checklist

- [ ] No crashes on startup (with or without mods)
- [ ] Recover gracefully from corrupt config files
- [ ] Handle missing OMSI 2 installation without undefined behavior
- [ ] Continue running even if a background thread crashes
- [ ] Validate all user input (file paths, config values, network packets)
- [ ] No memory leaks over 1+ hour of gameplay
- [ ] Save game state atomically (prevent data loss on crash)

### Testing for Safety

- **Fuzz Testing:**
  - Randomly generate malformed config files and test parsing
  - Generate corrupt archives and test extraction
  - Send invalid network packets in LAN play

- **Stress Testing:**
  - Run on lowest-spec hardware (integrated GPU, 4 GB RAM)
  - Test with largest mods (Ahlheim, complex add-ons)
  - Simulate running out of disk space mid-installation

---

## Security

### User Data Protection

- **Local Data:**
  - Settings and profiles are stored in `~/.openomsi/` (user-owned)
  - Never store plain-text passwords (if auth is added in future)
  - Hash check critical data (profile experience) to detect tampering

- **Network (LAN play):**
  - Validate all peer messages (range-check floats, whitelist commands)
  - Rate-limit: max 100 packets/second per peer
  - Drop oversized messages (> 64 KB)
  - Log suspected attacks: "Received invalid variable update from peer; connection rate-limited"

- **No Telemetry Without Consent:**
  - The presence service is opt-in (Settings → General → "Count me in the 'playing now'")
  - Only sends: random session ID, game version, OS type (no personal data)
  - Document this clearly in Settings

### Dependency Management

- **Regular Updates:**
  - Audit dependencies quarterly for security vulnerabilities
  - Use `cargo audit` in CI to block releases with known CVEs
  - Communicate security fixes to users clearly

- **Plugin Sandboxing (Future):**
  - If Lua plugins are expanded, consider sandboxing file access
  - Document what plugins can and cannot do

### Reporting Security Issues

- Add `SECURITY.md` (already exists; review and link from README)
- Ensure it's easy to find: link in top of README, website, Discord

---

## Implementation Priorities

### Phase 1 (Critical - v0.2.0)
1. ✅ Input validation for file paths and config files
2. ✅ User-friendly error messages
3. ✅ Streamlined first-time setup
4. ✅ Graceful handling of corrupt mods

### Phase 2 (Important - v0.3.0)
5. Progressive disclosure in settings
6. Accessibility: text size, high contrast
7. Built-in diagnostics (performance stats, crash logs)
8. Repair tool (`--verify-installation`)

### Phase 3 (Nice to Have - v0.4.0+)
9. Eye tracking support documentation
10. Fuzz testing in CI
11. Plugin sandboxing (if plugins expand)
12. Advanced telemetry options

---

## Checklist for PRs & Releases

Before merging code or shipping a release:

- [ ] **Does this add a new user-facing feature?** → Add user-friendly docs and tooltips
- [ ] **Does this change an error path?** → Is the error message user-friendly?
- [ ] **Does this touch file I/O?** → Is the path validated and errors handled?
- [ ] **Does this add network code?** → Is input validated and rate-limited?
- [ ] **Does this impact performance?** → Is it profiled and documented?
- [ ] **Does this add a new setting?** → Is it accessible and documented?
- [ ] **Does this break a previous workflow?** → Is there a migration path or clear docs?

---

## Questions & Discussion

**Future enhancement ideas:**
- Automatic graphics settings optimization (benchmark on first launch)
- Cloud save sync (encrypted, user-controlled)
- In-game tutorial mode
- Voice commands for accessibility
- Localization for more languages

