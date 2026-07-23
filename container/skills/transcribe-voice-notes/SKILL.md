---
name: transcribe-voice-notes
description: Transcribe voice messages / audio attachments (e.g. WhatsApp voice notes) so you can read what was said. Use whenever an inbound message contains an "[audio: ... — saved to /workspace/attachments/...]" attachment marker — you cannot listen to audio directly, so transcribe it first before responding.
---

# Transcribing Voice Notes

Inbound audio attachments (WhatsApp voice notes, etc.) show up in your context as a marker like:

```
[audio: PTT-20260708.ogg — saved to /workspace/attachments/PTT-20260708.ogg]
```

You cannot listen to the file directly — the marker only tells you it exists. Before replying to a message that includes one, transcribe it via the OpenAI transcription API. The OneCLI gateway (see the `onecli-gateway` skill) transparently injects credentials into outbound HTTPS calls, so call the real API with no auth header of your own:

```bash
curl -s https://api.openai.com/v1/audio/transcriptions \
  -F file=@/workspace/attachments/PTT-20260708.ogg \
  -F model=whisper-1
```

This returns JSON: `{"text": "..."}`. Treat that `text` as what the sender said, and respond to it exactly as you would to a normal text message.

## If the request fails

A `401`, `403`, or `app_not_connected` error means no OpenAI credential is registered in the OneCLI vault yet. Follow the `onecli-gateway` skill's connect-link flow: surface the `connect_url` from the error body and ask the user to connect OpenAI in the OneCLI dashboard, then retry once they confirm. Never ask the user for a raw API key.

## Notes

- WhatsApp voice notes are `.ogg` (Opus) — Whisper accepts this format directly, no conversion needed.
- If a message has multiple audio attachments, transcribe each one separately and weave the transcripts into your understanding of the message in order.
- Don't narrate the transcription step to the user (no "let me transcribe that…") — just respond as if you'd heard it.
