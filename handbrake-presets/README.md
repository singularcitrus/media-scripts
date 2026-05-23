# My handbrake presets

These are some of my handbrake presets, here's how to use them with the encode scripts

1. Install the HandBrake GUI (because the cli can't persist presets): `sudo apt install handbrake`
2. Import the preset in the handbrake UI (Action bar -> Presets -> Import Preset)
3. Call my encode script with `-g` to import from GUI and reference the name if the preset as it shows in the GUI


## H.265 NVENC 1080p - Passthru Audio - Remove Subtitles

Json file: `h265 cq22 sourcefpd passtru audio remove subs.json`

This preset is intended for compressing TV shows and movies for media servers such as Jellyfin or Plex while keeping processing time low through NVIDIA NVENC hardware acceleration.

### Video

| Setting | Value |
|----------|---------|
| Codec | H.265 (HEVC) NVENC |
| Quality Mode | Constant Quality |
| CQ Value | 22 |
| Encoder Preset | Slowest |
| Resolution | Preserves source up to 1080p |
| Framerate | Source (Variable Framerate) |
| HDR Metadata | Preserved when present |

The preset uses H.265/HEVC with NVENC hardware encoding and a Constant Quality value of 22, aiming for a balance between file size reduction and visual quality. The encoder is configured with the slowest NVENC preset to maximize compression efficiency.

### Audio

| Setting | Value |
|----------|---------|
| Audio Handling | Passthrough |
| Track Selection | All audio tracks |
| Fallback Encoder | None |

All audio tracks are copied directly from the source whenever possible, preserving original quality and avoiding unnecessary transcoding.

### Subtitles

| Setting | Value |
|----------|---------|
| Subtitle Handling | Remove all subtitles |
| Burned Subtitles | Disabled |
| Foreign Audio Search | Disabled |

No subtitle tracks are included in the output file. This helps reduce file size and avoids carrying unwanted subtitle streams.

### Metadata

| Setting | Value |
|----------|---------|
| Chapter Markers | Preserved |
| Media Metadata | Preserved |

Chapter information and file metadata are retained in the encoded output.

### Container

| Setting | Value |
|----------|---------|
| Format | MP4 |

Output files are written to an MP4 container for broad compatibility across devices and media servers.

### Intended Use

- Jellyfin or Plex media libraries
- TV show collections
- Movie archives
- Batch compression of existing H.264/H.265 content
- Situations where encoding speed is more important than absolute compression efficiency

### Trade-offs

| Advantage | Drawback |
|------------|------------|
| Fast hardware encoding | Larger files than x265 CPU encoding |
| Audio quality preserved | Audio tracks may remain large |
| Reduced storage usage | Subtitle tracks are permanently removed |
| Broad MP4 compatibility | Limited to 1080p output profile |