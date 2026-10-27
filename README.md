# Clarion MCP

A Claude Code plugin that turns Uzbek speech into text. It transcribes
audio and video into plain text, timestamped segments, or SRT/VTT
subtitles. It runs on the [UzASR](https://api.uzasr.uz) API: a GigaAM
model fine-tuned for Uzbek, decoded with a KenLM language model.

The plugin has two parts:

- **MCP server** at `https://api.uzasr.uz/mcp`. Your API key is asked for
  once at install time and kept in the OS keychain.
- **`/clarion-mcp:transcribe` skill.** It checks the file size,
  compresses the file with ffmpeg when needed, uploads it, transcribes it,
  and writes the result to a file.

## Installation

In Claude Code:

```
/plugin marketplace add rustammdev/clarion-mcp
/plugin install clarion-mcp@clarion-mcp
```

You will be asked for a UzASR API key (`uzasr_live_…`). To change it
later, run `/plugin configure clarion-mcp@clarion-mcp`. Create a separate
key for this plugin, so you can revoke it without affecting your other
integrations.

## Usage

Just ask in plain words and the skill starts on its own:

- "Turn lesson.mp4 into Uzbek subtitles and save them to lesson.srt"
- "Transcribe interview.mp3 and summarise the key points"
- "Transcribe every mp3 in the podcasts/ folder"

Or call it directly: `/clarion-mcp:transcribe lesson.mp4 srt`.

| Formats | Description |
|---|---|
| `text` | plain transcript |
| `segments` | timestamped lines |
| `srt`, `vtt` | subtitles, with cues built from word timings |

| Models | Description |
|---|---|
| `uzasr-large` | default, most accurate |
| `uzasr-base` | faster and cheaper |

## Limits

- Files up to 25 MB, audio up to 3 hours.
- Up to 10 uploads per key may wait for transcription at a time.
- Upload and download links work once and expire after 15 minutes.
- Only Uzbek speech is supported.

## Security and privacy

- Without asking, the skill may call only the Clarion MCP tools. `curl`
  and `ffmpeg` go through Claude Code's normal permission prompts.
- Audio stays on the server only while it is being processed and is then
  deleted. Transcripts are deleted after 7 days.
- With `save_to`, the transcript goes straight into a file and never
  enters the agent's context.

## Development

```
claude plugin validate . --strict
claude --plugin-dir .
```

## License

[MIT](LICENSE)
