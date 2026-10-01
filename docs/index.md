# py_premiere

`py_premiere` is a Python package for working with Adobe Premiere Pro project
files (`.prproj`). It is the Premiere sibling of
[py-aep](https://github.com/forticheprod/py-aep).

A `.prproj` file is a gzip-compressed XML object graph. `py_premiere` parses
it into typed Python classes mirroring the
[Premiere Scripting Guide](https://ppro-scripting.docsforadobe.dev/) object
model, and saves it back with **byte fidelity**: parsing a project and saving
it unchanged reproduces the original file byte for byte.

## Features

- Read `.prproj` files: project item tree (bins, clips, media paths),
  sequences, tracks, track items with timeline and source times
- Modify read/write attributes (validated inputs) and save with
  `Project.save(path)`
- Byte-identical round-trip; unsupported constructs are refused at parse
  time instead of silently corrupted on save
- Values validated against Premiere's own ExtendScript DOM (ground-truth
  JSON fixtures)
- CLI tools: `pr-inspect`, `pr-compare`, `pr-visualize`, `pr-validate`

## Installation

```sh
uv add py-premiere
# or
pip install py-premiere
```

## Getting started

```python
import py_premiere

app = py_premiere.parse("myproject.prproj")
project = app.project

for sequence in project.sequences:
    print(sequence.name, sequence.frame_size)
    for track in sequence.video_tracks:
        for clip in track.clips:
            print(f"  {clip.name}: {clip.start.seconds:.2f}s - {clip.end.seconds:.2f}s")

sequence = project.sequences[0]
sequence.name = "Renamed"
project.save("renamed.prproj")
```

