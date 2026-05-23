# Media Encoding Tools

Small terminal tools for batch-encoding media with HandBrakeCLI and checking how much space you actually saved afterwards. Because apparently staring at shrinking file sizes is a valid lifestyle choice.

This repo contains two scripts:

- `encode_folder` — recursively encodes video files from one folder tree into another, preserving the relative folder structure.
- `compression_stats` — compares an original folder tree against a compressed folder tree and reports file-by-file, folder-level, and total savings.
- `engaudiostripsubs`— remuxes MKV files in the current directory, keeping only English audio tracks and removing all subtitle tracks.

_There are others, but they are quick utility scripts that happened to be in the same folder as the main 2, use at your own risk, and your mileage may vary. I am not going to document them unless they become proper tools_

## Requirements

### Required

- Linux (_or unix if you like to live on the edge, although it probably won't work right on unix_)
- Bash
- `HandBrakeCLI`
- `mkvmerge` (MKVToolNix)
- GNU/coreutils-style tools such as:
  - `find`
  - `sort`
  - `stat`
  - `realpath`
  - `numfmt`
  - `awk`
  - `tput`

On Debian/Ubuntu-based systems, HandBrakeCLI can usually be installed with:

```bash
sudo apt install handbrake-cli
```

The scripts are written with Linux in mind. They may need small changes on macOS because BSD `stat` and GNU `stat` enjoy being pointlessly different, as is tradition.

## `encode_folder`

Batch-encodes media files under an input directory using a HandBrake preset, writing the encoded files to an output directory while preserving the original relative folder structure.

### Basic usage

```bash
./encode_folder -p "Preset Name" -i "/path/to/input" -o "/path/to/output"
```

Example for a movies folder:

```bash
./encode_folder \
  -p "H.265 MKV 1080p30" \
  -i "/media/Movies" \
  -o "/encoded/Movies"
```

Example for a series folder:

```bash
./encode_folder \
  -p "H.265 MKV 1080p30" \
  -i "/media/Series/Some Series" \
  -o "/encoded/Series/Some Series"
```

### Folder structure preservation

Given this input:

```text
Input/
└── Season 04/
    ├── Episode 01.mkv
    └── Episode 02.mkv
```

The output becomes:

```text
Output/
└── Season 04/
    ├── Episode 01.mkv
    └── Episode 02.mkv
```

For movies:

```text
Movies/
└── Some Movie/
    └── Some Movie.mkv
```

becomes:

```text
Encoded Movies/
└── Some Movie/
    └── Some Movie.mkv
```

### Options

```text
-p <preset>       HandBrake preset name
-i <input_dir>    Input media folder
-o <output_dir>   Output media folder
-e <ext>          Output file extension, default: mkv
-g                Pass --preset-import-gui to HandBrakeCLI
-f                Force re-encode even if output already exists
```

### Supported input extensions

`encode_folder` currently looks for:

```text
.mkv .mp4 .avi .m4v .mov .ts .wmv
```

### Output extension

By default, output files use `.mkv`.

To output `.mp4` instead:

```bash
./encode_folder \
  -p "Fast 1080p30" \
  -i "/path/to/input" \
  -o "/path/to/output" \
  -e mp4
```

### Skipping existing files

The script skips files when an output file with the same base name already exists in the target folder, regardless of extension.

For example, if this exists:

```text
Output/Season 01/Episode 01.mkv
```

then this input will be skipped:

```text
Input/Season 01/Episode 01.mp4
```

Use `-f` to force re-encoding:

```bash
./encode_folder -p "Fast 1080p30" -i Input -o Output -f
```

### Live progress display

While encoding, the script shows HandBrake progress on a single terminal line, including:

- current encoding percentage
- current output file size
- projected final output size
- estimated percentage of original size
- estimated saving percentage

Example:

```text
Encoding: task 1 of 1, 42.10 % (...) | File Size: 132MB | Projected: 314MB | Est: 19.1% of original, save 80.9%
```

The projection is an estimate based on current output size divided by current progress percentage. It gets more useful later in the encode. Early values can be hilariously wrong because video encoding is not legally required to make you happy.

### Logs

A log is written to:

```text
<output_dir>/encode.log
```

The terminal only shows cleaned progress output, but the log captures the HandBrake progress lines that pass through the progress filter.

### Ctrl+C behaviour

If interrupted with Ctrl+C, the script:

- clears the progress line
- reports the interruption
- removes the partial output file for the current encode
- exits completely instead of continuing to the next file

This avoids leaving half-written files around like tiny digital landmines.

## `compression_stats`

Compares an original media folder against a compressed media folder and reports savings.

It is read-only. It does not modify files.

### Basic usage

```bash
./compression_stats /path/to/original /path/to/compressed
```

Example:

```bash
./compression_stats \
  "/media/Series/Some Series" \
  "/encoded/Series/Some Series"
```

### What it compares

The script walks the original folder tree recursively and tries to find matching files in the compressed folder tree.

It first checks for the exact same relative path. If that is not found, it falls back to finding a file with the same base name in the same relative folder, even if the extension changed.

For example, this original:

```text
Original/Season 04/Episode 01.mp4
```

can match this compressed file:

```text
Compressed/Season 04/Episode 01.mkv
```

Humanity survived another extension change. Barely.

### Supported media extensions

`compression_stats` currently considers:

```text
mp4 mkv avi m4v mov ts wmv webm
```

### Output

The report includes:

- per-file original and compressed sizes
- per-file savings
- missing compressed files
- grouped folder summaries
- total original size
- total saved so far
- projected saving if only some files are compressed
- final total size if all files are compressed

### CSV export

Generate a CSV report:

```bash
./compression_stats --csv report.csv /path/to/original /path/to/compressed
```

This writes a CSV file with file-level comparison data.

### Read from CSV

Replay a saved CSV report:

```bash
./compression_stats --from-csv report.csv
```

Useful if you want to view the report again without rescanning the folders.

## engaudiostripsubs

Remuxes all MKV files in the current directory, keeping only English audio tracks and removing all subtitle tracks.

The original file is temporarily renamed during processing and replaced with the remuxed version.

### Usage

```bash
cd /path/to/videos
./engaudiostripsubs
```

### What it does

For every `.mkv` file in the current directory:

- Keeps only audio tracks with language set to `eng`
- Removes all subtitle tracks
- Preserves video streams
- Replaces the original file with the remuxed version

### Example

Before:

```text
Movie.mkv
├── Video (H.265)
├── Audio (English)
├── Audio (French)
├── Subtitle (English)
└── Subtitle (Spanish)
```

After:

```text
Movie.mkv
├── Video (H.265)
└── Audio (English)
```

> [!CAUTION]
> This script modifies files in-place. Ensure you have backups or test on a small sample before processing a large library.

## Suggested workflow

Encode into a separate output folder first:

```bash
./encode_folder \
  -p "H.265 MKV 1080p30" \
  -i "/media/Movies" \
  -o "/encoded/Movies"
```

Then compare the original and encoded trees:

```bash
./compression_stats \
  "/media/Movies" \
  "/encoded/Movies"
```

Optionally save a CSV:

```bash
./compression_stats \
  --csv movies-compression-report.csv \
  "/media/Movies" \
  "/encoded/Movies"
```

After verifying the output, manually replace or archive the originals using whatever level of paranoia your storage situation deserves.

## Safety notes

`encode_folder` writes to the output directory you provide. It does not intentionally modify the original input files.

Still, do the sensible thing:

- encode into a separate output folder
- verify the results
- check subtitles, audio tracks, and quality before deleting anything
- keep backups of anything you care about

The scripts are useful, not magical. Bash is still Bash, which is basically a haunted vending machine that accepts file paths.

## Use of AI
Since I don't like it when developers don't declare their AI use, here goes:

Yes, I used AI to help me in the development of these scripts, since bash is far from my strong suit. Rest assured, I've run these scripts in my own media environment and have trusted it with compressing my files and reporting back. I have also replaced some of my only copies of shows with the compressed version generated with my scripts.