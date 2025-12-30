# Audio Assets

The format of choice for all audio assets will be: `opus`.

Audio Expectations:
* Volume Normalized


## Example FFMPEG command

> `ffmpeg -i input.mp3 -af loudnorm=I=-16:TP=-1.5:LRA=11 -c:a libopus -b:a 128k output.opus`

### Breakdown:
* `-i input.mp3` — input file (works with any audio format)
* `-af loudnorm=I=-16:TP=-1.5:LRA=11` — EBU R128 loudness normalization
  * `I=-16` — target integrated loudness (LUFS), -16 is good for streaming
  * `TP=-1.5` — true peak limit (dB), prevents clipping
  * `LRA=11` — loudness range target
* `-c:a libopus` — encode with Opus codec
* `-b:a 128k` — bitrate (adjust as needed: 64k for speech, 128k-192k for music)
