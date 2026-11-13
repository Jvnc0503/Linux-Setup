# Dev Tools

## Latex

```bash
sudo apt install texlive-latex-base texlive-latex-recommended texlive-latex-extra texlive-fonts-recommended texlive-xetex texlive-luatex texlive-publishers texlive-lang-spanish chktex latexmk
```

## yt-dlp

```bash
sudo apt install yt-dlp
```

### JS runtime

```bash
curl -fsSL https://deno.land/install.sh | sh
```
## Whipper (CD ripping)

### 1. Install Whipper and Dependencies
```bash
sudo apt update
sudo apt install whipper cd-paranoia cdrdao flac sox python3-musicbrainzngs python3-mutagen python3-cdio
```
> *Verification:* Run whipper --version in your terminal to confirm the executable is installed and returns a version number without Python module errors.

### 2. Calibrate Your Optical Drive (One-Time Setup)
Insert a commercial, widely pressed audio CD (not a burned CD-R) into your drive. Popular retail albums have the highest likelihood of existing in the AccurateRip database.

1. Identify your drive:
  ```bash
  whipper drive list
  ```
  > Note the exact vendor and model strings printed in the output.

2. Analyze drive caching behavior:
  Optical drives cache audio data in memory, which can mask read errors during re-reads. Test whether `cd-paranoia` can defeat your drive's cache:
  ```bash
  whipper drive analyze
  ```
  > *Verification:* Look for `INFO:whipper.command.drive:adding drive cache behaviour to configuration file`.

3. Find and save your drive's read offset:
  Every optical drive model reads audio data shifted by a fixed number of samples. Look up your drive's model in the [AccurateRip Drive Offset Database](https://www.accuraterip.com/driveoffsets.htm), then verify and save it (replace `OFFSET` with the numeric value, omitting any `+` sign, e.g., `6` or `-102`):
  ```bash
  whipper offset find -o OFFSET
  ```
  (If your drive is not listed in the database, run `whipper offset find` without `-o` to automatically test common offsets).
  > *Verification:* Confirm the terminal outputs `Read offset of device is: OFFSET` followed by `adding read offset to configuration file`.

### 3. Configure Defaults
Whipper automatically creates `~/.config/whipper/whipper.conf` during calibration. Open it to set clean folder templates and lossless FLAC defaults so you do not have to pass long flags every time:
```toml
[main]
path_filter_dot = True
path_filter_posix = True
path_filter_vfat = False

[whipper.cd.rip]
output_directory = ~/Music/Rips
disc_template = %A/%d (%y)/%A - %d
track_template = %A/%d (%y)/%t - %n
cover_art = complete
max_retries = 5
continue_on_error = True
```
- `cover_art = complete` downloads the album cover from Cover Art Archive, saves it as a standalone image file in the album folder, and embeds it into each .flac file.
- `max_retries=N`: Sets the maximum number of times whipper will re-read a track if the Test and Copy CRCs do not match (the default is `5`). Setting a finite limit prevents whipper from looping indefinitely on a scratched sector.
- `continue_on_error`: Tells whipper not to abort the entire session if a track still fails CRC verification after exhausting `max_retries` (or encounters an unrecoverable read error). It saves the best attempt, logs the failure/mismatch in the `.log` file, and proceeds to the next track.
> *Verification:* Run cat `~/.config/whipper/whipper.conf` to verify that both your `[drive:...]` section (with `read_offset` and `defeats_cache`) and your `[whipper.cd.rip]` preferences are present.

### 4. Rip a CD
With your drive calibrated, insert any audio CD and run:
```bash
whipper cd rip
```
> (If you have multiple optical drives, specify the device before `rip`: `whipper cd -d /dev/sr1` rip.)

Whipper will automatically:
1. Query MusicBrainz for release metadata (if multiple pressings match, it will prompt you to select the exact release ID).
2. Detect the disc's Table of Contents (TOC), pre-gaps, ISRCs, and any Hidden Track One Audio (HTOA).
3. Perform a Test & Copy rip of every track, comparing the CRCs of two independent reads.
4. Encode the tracks to FLAC and verify the checksums against the AccurateRip online database.

> *Verification:* Inspect the generated `.log` file in your output directory. Every track should show matching `Test CRC` and `Copy CRC` hashes alongside `Accurately ripped` (with a confidence number).
