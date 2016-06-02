## Porting ノラと皇女と野良猫ハート (AKA Nora to Oujo to Noraneko Heart, AKA Noratoto)

### Requirements

* [whale-tools]
* [arc-unpacker]
TODO

### Versions available

There are at least 4 known versions of the game in the wild: v1.00
(original golden release), v1.01, v1.02, v1.03 (downloadable as patches
from official website).

v1.03 includes the following asset archive files:

* script.dat - scripts
* arc0.dat - everything
* arc1.dat - voice sound clips
* arc2.dat - FullHD graphics
* arc3.dat - contents of patch

### Basic porting

The main problem is that Whale format archives do not include any
filenames, only the hash sums calculated from the file names. Thus, one
must have a list of file names to know which binary chunk identified by a
given hash corresponds to which file name. On the other hand, it's only
possible to do so by looking up file names used in VN scripts.

So, ideally it goes like that:

1. Unpack Whale scripts using [whale-tools]; `archiver` script should be
edited before running, replacing line with `GAME_TITLE` with:

```python
GAME_TITLE = "ノラと皇女と野良猫ハート"
```

Whale scripts should be converted to UTF-8 (as they're originally in
ShiftJIS).

This step can be achieved by running `./noratoto-extract-scripts`

2. Now we need create a map of scripts to its IDs (i.e. that
`00022_900a40a6e5662415.script` is actually `A4_01a`). Such a map can be
created semi-automatically by analyzing labels inside Whale script files,
however, fine-tuning by hand is advised to remove all non-gameplay
related scripts (i.e. menu, titles, etc).

This step can be achieved by running `./noratoto-gen-script-list` and
editing `script-list.txt` in noratoto-trans.

3. Generate list of files used in scripts by running
`./noratoto-gen-files-list`. This generates `files-list.txt` in
noratoto-trans.

4. Extract files by the file list, again, using [whale-tools] `archiver`.
This step is automated by running `./noratoto-extract-files`.

5. Convert all possible TLG images into PNG images. This step requires
[arc-unpacker] to do the actual conversion - be sure to set up path to it
in `config` file. Automated conversion is done by running `./noratoto-tlg2png`

6. TLG images actually come in 2 versions: TLG6 images (which are
basically normal bitmaps) and TLG0 containers, which are actually images
+ some info on proper way to achieve properly composited images from
pieces. We need to extract this compositing info in separate image
description file - `imgs.json`. This task is achieved by running
`./noratoto-process-tlg0`.

Note: this step requires library from [story_utils] and TLG0 format
description in .ksy compiled as .rb module.

7. Finally, everything is ready to convert Whale scripts to .story
format. This is performed by [whale2story] utility, which can be launched
automatically on every Whale script by running `./noratoto-script-to-story`.

[whale-tools]: https://github.com/vn-tools/whale-tools
[arc-unpacker]: https://github.com/vn-tools/arc_unpacker
[story_utils]: https://github.com/mnakamura1337/story_utils
