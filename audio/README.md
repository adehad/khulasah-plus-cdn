# Audio Assets

Audio is served in two formats depending on file size:

* **Single `.opus` file** — for assets under 19MB, served directly via jsdelivr
* **HLS stream** — for assets >= 19MB, split into fMP4 segments to stay within jsdelivr's 20MB per-file limit. Played back via hls.js

All audio is volume-normalized to EBU R128 standards.

## Processing Pipeline

### Stage 1: MP3 to Opus

Converts all source files to Opus format.

> `ffmpeg -i input.mp3 -af loudnorm=I=-16:TP=-1.5:LRA=11 -c:a libopus -b:a 128k output.opus`

#### Breakdown:
* `-i input.mp3` — input file (works with any audio format)
* `-af loudnorm=I=-16:TP=-1.5:LRA=11` — EBU R128 loudness normalization
  * `I=-16` — target integrated loudness (LUFS), -16 is good for streaming
  * `TP=-1.5` — true peak limit (dB), prevents clipping
  * `LRA=11` — loudness range target
* `-c:a libopus` — encode with Opus codec
* `-b:a 128k` — bitrate (adjust as needed: 64k for speech, 128k-192k for music)

### Stage 2: Opus to HLS (files >= 19MB only)

Any `.opus` file at or above 19MB is split into an HLS stream.
The Opus audio is stream-copied (not re-encoded) into fMP4 segments.

> `ffmpeg -i input.opus -c:a copy -f hls -hls_time 10 -hls_list_size 0 -hls_segment_type fmp4 -hls_fmp4_init_filename init.mp4 -hls_segment_filename "output/seg%03d.m4s" output/playlist.m3u8`

#### Breakdown:
* `-i input.opus` — the normalized Opus file from Stage 1
* `-c:a copy` — stream-copy the audio without re-encoding (lossless, fast)
* `-f hls` — HLS (HTTP Live Streaming) output format
* `-hls_time 10` — split audio into 10-second segments
* `-hls_list_size 0` — include every segment in the playlist file
* `-hls_segment_type fmp4` — use fragmented MP4 containers instead of MPEG-TS
* `-hls_fmp4_init_filename init.mp4` — initialization segment for the fMP4 stream
* `-hls_segment_filename "output/seg%03d.m4s"` — naming pattern for segment files
* `output/playlist.m3u8` — the HLS playlist file

The original `.opus` file is deleted after a successful split.
