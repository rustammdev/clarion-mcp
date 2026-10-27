# Clarion MCP

A Claude Code plugin that turns Uzbek speech into text. It transcribes
audio and video into plain text, timestamped segments, or SRT/VTT
subtitles. It runs on the [UzASR](https://api.uzasr.uz) API: a GigaAM
model fine-tuned for Uzbek, decoded with a KenLM language model.

The plugin has two parts:

- **MCP server** at `https://api.uzasr.uz/mcp`. No API key to copy: you
  sign in to Clarion in the browser once, and Claude Code keeps the token.
- **`/clarion-mcp:transcribe` skill.** It checks the file size,
  compresses the file with ffmpeg when needed, uploads it, transcribes it,
  and writes the result to a file.

## Installation

In Claude Code:

```
/plugin marketplace add rustammdev/clarion-mcp
/plugin install clarion-mcp@clarion-mcp
```

On first use, run `/mcp`, pick **clarion** and choose **Authenticate**. A
browser opens: sign in to clarion.uz and click **Allow**. Claude Code stores
the token in the OS keychain and refreshes it on its own. Connected apps
are listed in the Clarion console, where you can disconnect them any time.

For CI or headless use, add the server with an API key instead:

```
claude mcp add --transport http clarion https://api.uzasr.uz/mcp \
  --header "Authorization: Bearer $UZASR_API_KEY"
```

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
- Up to 10 uploads per account may wait for transcription at a time.
- Upload and download links work once and expire after 15 minutes.
- Only Uzbek speech is supported.

## Security and privacy

- Sign-in uses OAuth 2.1 with PKCE. Access tokens live for an hour and
  refresh tokens rotate on every use; no long-lived key sits on disk.
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
