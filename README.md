# py_premiere
`py_premiere` is a Python package for working with Adobe Premiere Pro project files (.prproj).

<center><strong><a href="https://Pouf.github.io/py-premiere/">Explore the docs »</a></strong></center>

## About

Premiere Pro project files are gzip-compressed XML object graphs. This
package parses them into an `Application` object containing the project,
its item tree, sequences, tracks and clips. The API is very close to the
[ExtendScript API](https://ppro-scripting.docsforadobe.dev/), with
validated inputs.

Sibling project of [py-aep](https://github.com/forticheprod/py-aep).

## Features

### Supported
* Reading `.prproj` files: project panel tree (bins, clips, media paths),
  sequences, tracks, track items (timeline and source times, playback-speed
  aware)
* Components (effects) with parameters, static values and keyframes
  (materialized ones - Premiere synthesizes untouched intrinsics at runtime
  and stores nothing for them)
* Track transitions (name, matchName, timeline times, the cut point they
  cover, and the wipe controls: border width/colour, direction, reverse,
  anti-aliasing) - a surface the ExtendScript DOM never exposed
* Sequence and clip/project item markers (read and write: name, comments,
  start/end, colour index)
* Footage interpretation (alpha usage, ignore/invert alpha, field type,
  pixel aspect - anamorphic media included), media start timecode
  (`start_time`) and attached proxies (`has_proxy`, `proxy_path`)
* Track state: mute, lock and sync lock
* Sequence work area, in/out points and playhead position
* Retiming: playback speed, reverse, and how a retimed clip fills frames
  (frame sampling / blending / optical flow)
* Generated media - Black Video, colour mattes, bars and tone, transparent
  video, counting leaders - identified by kind rather than by an empty path
* Creating from scratch: `py_premiere.new()` projects, `add_sequence`,
  `add_bin`, `import_files` (BMP/PNG/JPEG/GIF/TIFF/PSD stills; WAV, AIFF,
  M4A, MP3, AAC and WMA audio; AVI, H.264/H.265/ProRes/DNxHR in MP4/MOV and MPEG-2 in
  MXF, carrying start timecode and field dominance) and placing clips with
  `add_clip` / `insert_clip`
* Modifying read/write attributes (names, track item times, markers) and
  saving with `Project.save(path)` (atomic, refuses to overwrite)
* Adding and removing transitions (`track.add_transition`), on a clip end or
  on a cut - clamped to the available handles exactly as Premiere does, and
  re-saved unchanged by it
* Byte-identical round-trip: parse then save reproduces the original file
  byte for byte; files the serializer cannot faithfully reproduce are
  refused at parse time instead of silently rewritten
* Values validated against Premiere's own ExtendScript DOM (ground-truth
  JSON fixtures exported from Premiere)
* Effect masks (on an effect or on the clip) and time remapping - surfaces
  the ExtendScript DOM never exposed
* Caption tracks: each caption's text and both its timeline and source
  times (`sequence.caption_tracks`)
* Subclips (`is_subclip` and the boundaries that narrow them) and Motion
  Graphics template clips (their Essential Graphics parameters, including
  the edited text)
* FCPXML export (`py_premiere.export.export_fcpxml`)
* CLI tools: `pr-inspect` (structure), `pr-compare` (resave-churn-aware
  diff), `pr-visualize` (parsed tree), `pr-validate` (ground truth)

### Not supported (yet)
* Popup/menu param value decoding (exposed as raw strings; verified
  encodings are decoded). Arbitrary-data params decode when they hold text
  or a shape path
* The caption enable flags and stroke-alignment popup (their slots are
  decoded but the values are all 0/1/2, so nothing names them yet)
* Importing media beyond the formats listed above

XMP metadata is not a gap: Premiere writes an item's XMP packet into the
media file, not into the project, so there is none in a `.prproj` to read.

## Installation

### uv (recommended)
```sh
uv add py-premiere
```

### pip
```sh
pip install py-premiere
```

## Getting started

```python
import py_premiere

app = py_premiere.parse("myproject.prproj")
project = app.project

for sequence in project.sequences:
    for track in sequence.video_tracks:
        for clip in track.clips:
            print(clip.name, clip.start.seconds, clip.end.seconds)

project.sequences[0].name = "Renamed"
project.save("renamed.prproj")
```
