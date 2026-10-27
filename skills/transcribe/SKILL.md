---
name: transcribe
description: Transcribe Uzbek speech from local audio or video files into text, timestamped segments, or SRT/VTT subtitles using the Clarion MCP tools. Use when the user asks to transcribe, caption or subtitle Uzbek audio/video, or to turn a recording into text (o'zbekcha transkript, matnga o'girish, subtitr, расшифровка узбекской речи).
argument-hint: "<file or folder> [text|segments|srt|vtt]"
allowed-tools: mcp__plugin_clarion-mcp_clarion__list_models, mcp__plugin_clarion-mcp_clarion__create_upload, mcp__plugin_clarion-mcp_clarion__transcribe, mcp__plugin_clarion-mcp_clarion__get_transcription
---

# Uzbek transcription with Clarion

The `clarion` MCP server (UzASR API) only understands Uzbek speech. Files
never pass through tool arguments: they go up and come back with one-time
curl commands that the tools return.

## Steps for each file

1. **Check the file.** Get its size (`wc -c < file`). The upload limit is
   25 MB. For video, or for audio over the limit, extract compressed mono
   audio first and upload that instead:
   `ffmpeg -i input.mp4 -vn -ac 1 -ar 16000 -c:a libopus -b:a 32k input.ogg`
   (about 0.25 MB per minute; audio up to 3 hours is accepted).
2. **Upload.** Call `create_upload` with the file path, then run the
   returned curl command exactly as given, from the directory the path is
   relative to. It prints `{"upload_id": ..., "bytes": ...}` on success.
3. **Transcribe.** Call `transcribe` with the `upload_id`:
   - subtitles: `format` `srt` or `vtt`, and `save_to` set to the source
     name with the new extension (`lesson.mp4` → `lesson.srt`);
   - audio longer than about 10 minutes, or when the user wants a file:
     `save_to` with `format` `text` or `segments`;
   - short audio the user wants to read: no `save_to`, the text comes back
     inline.
   Model: `uzasr-large` (default, most accurate) or `uzasr-base` (faster,
   cheaper).
4. **Wait if needed.** "Still transcribing" means the job continues on the
   server: call `get_transcription` with the given `job_id` and the same
   `format` and `save_to` until it finishes.
5. **Save.** With `save_to`, run the returned curl command, then confirm
   that the file exists and is not empty. Report the path, the audio
   duration and the word count.

## Several files

At most 10 uploads per key may wait for `transcribe`. Work in batches of
up to 10: upload a file, start `transcribe`, move on, and collect results
with `get_transcription`.

## Errors

| Error code | What to do |
|---|---|
| `upload_not_found` | The upload expired (15 min) or was never finished: call `create_upload` again |
| `file_too_large` | Compress with the ffmpeg command above |
| `concurrency_limit_exceeded` | Transcribe the uploads already waiting first |
| `model_unavailable`, `model_overloaded` | The GPU is starting or busy: wait about 10 s and retry |
| `audio_unreadable` | Convert with ffmpeg and upload again |
| `download_not_found` | The link was used or expired: call `get_transcription` with `save_to` again |
| `invalid_api_key` | Ask the user to set the key with `/plugin configure clarion-mcp@clarion-mcp` |

## Rules

- Each curl command and its token work once. Never show tokens to the user
  and never reuse or edit the commands.
- Do not paste long transcripts into the chat. Save them to a file and
  summarise if the user asks for that.
- Only Uzbek is supported. For other languages, tell the user that this
  tool does not fit.
