# PatchPeek

Pull a mobile game's files off an Android emulator, unpack them, and see
exactly what each update added — including artwork — before it shows up in
game.

## What it does

Live-service games ship a thin APK and download the rest. New content usually
lands on your device days or weeks before the switch is flipped, so the way to
find it is to snapshot the files after every patch and diff the snapshots.

PatchPeek does that whole loop:

1. **Pull** — finds your running BlueStacks instance over adb and copies the
   APK, its splits, the OBB, and `Android/data/<package>/files` (the
   downloaded content, which is where the interesting things live).
2. **Extract** — unpacks every archive, then opens the Unity bundles and
   exports textures, sprites, TextAssets and MonoBehaviour data.
3. **Snapshot** — hashes everything and stores a normalized copy of every text
   file.
4. **Diff** — compares against the previous snapshot and reports every file and
   every changed line, with localization and config tables sorted to the top.
5. **Browse** — a built-in viewer: filter, search, preview images on a
   transparency checkerboard, read prettified JSON, tick what you want and
   export it. Every capture keeps two views — the **full asset list** and each
   **diff** — side by side in the `view` dropdown, so comparing never costs you
   the ability to scroll the whole thing.

   Requires Python 3.9+ with tkinter-free PyQt6 (the python.org installer is
fine). You also need `adb` — BlueStacks ships one as `HD-Adb.exe` and
PatchPeek finds it automatically, or install Android platform-tools.

## Using it

1. Start BlueStacks and turn on **Settings → Advanced → Android Debug Bridge**.
2. Launch the game and let it finish downloading content. PatchPeek reads what
   is on disk; it cannot make the game fetch anything.
3. Press **Find BlueStacks**, then **Capture snapshot now**.

The first capture has nothing to compare against, so it indexes everything as a
baseline. After the next update, capture again and the **New assets** tab shows
only what changed — green for new, amber for changed.

### Things worth knowing

- **Content can change without the version changing.** Publishers push assets
  to the CDN without shipping a new APK. Capture whenever you suspect a drop;
  captures of the same version get a timestamp suffix so nothing is overwritten.
- **Direction matters.** The `compare from … to …` picker controls which way
  the diff runs. "New" means *in the `to` capture but not the `from` one*, so
  if you capture an older build second, use the picker rather than the default.
- **Keep raw files** keeps the extracted assets on disk so the image viewer has
  something to show. It costs disk space; clear out `extracted/` when done.
- A full capture takes a while — most of it is opening several thousand Unity
  bundles. The progress bar reports the phase and an ETA.
- **A diff only shows what moved between two captures.** Content that was
  already sitting unreleased in both looks unchanged, because it is. To find
  the backlog, use **first seen**: press *Rebuild first-seen*, then sort by
  that column. An asset that first appeared two captures ago and still isn't
  live in game is the highest-signal thing in a dump.
- Snapshots are never overwritten. Re-capturing a version you already have
  saves alongside it with a timestamp suffix.

## Speed

Snapshotting is dominated by per-file overhead on Windows, not by hashing
(SHA-256 of 188 MB across 4,040 files takes about half a second). So:

- normalized text goes into **one zip per snapshot** instead of tens of
  thousands of loose files
- reads and hashes run on a **thread pool**, because those calls are
  latency-bound on NTFS once a realtime AV scanner is in the path
- Unity bundles are opened in a **process pool** — that part really is
  CPU-bound
- files placed in the gallery are **hardlinked**, not copied

##Requirements

Python 3.9+
PyQt6>=6.6
UnityPy>=1.20
Pillow>=10.0
