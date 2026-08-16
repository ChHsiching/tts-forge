---
name: tts-forge
description: Provider-agnostic voiceover production — pick a TTS provider, audition voices for free, keep spoken scripts and display text separate, then synthesize incrementally with cost transparency. Use when a video has no audio track and needs a voice, when the user mentions 配音 / TTS / voiceover / 合成语音 / narrate, when a voice must be chosen or swapped, or when another skill (e.g. a video router) hands off a voiceover task. Covers MiniMax (mmx-cli) and OpenAI built-in, plus paste-in snippets for ElevenLabs / edge-tts / Azure / Google.
---

# tts-forge

Turn display text into a narrated audio track: choose a provider, choose a voice by ear, then synthesize without burning money twice.

## Step 1 — Choose the provider with the user

Present the trade, let them pick:

| Route | Best at | Cost model | Setup |
|---|---|---|---|
| MiniMax (`mmx-cli`) | Chinese narration, natural prosody | ~¥0.5 / 1k chars; requires balance | `mmx` CLI + login (`--region=cn`) |
| OpenAI TTS | quick start, most agents already hold a key | per-char, cheap | `OPENAI_API_KEY` |
| edge-tts | free, offline-ish drafts | free | local binary |
| ElevenLabs / Azure / Google | premium English/brand voices | subscription | provider portal |

Two provider-specific traps worth stating to the user up front: MiniMax authentication is not enough — the account needs balance topped up; and the default MiniMax voice is unpleasant enough that voice selection (Step 2) is mandatory, not optional.

After the user picks, collect the credential (API key / login) before any synthesis attempt, and store it where the run's scripts expect it (env var or CLI session).

## Step 2 — Audition voices before spending

Never synthesize the full script to "hear how it sounds". Audition on the provider's free channel:

- MiniMax: the voice library page (minimaxi.com/audio/voices) previews every voice free — pick there, then bring the voice ID here. Do not burn synthesis quota on auditions.
- OpenAI: synthesize one short line per candidate voice (a few cents total) and play them for the user.

The user picks the voice. Record the voice ID in the run notes — every later synthesis command takes it explicitly.

## Step 3 — Spoken script vs display text

One script, two renderings — they are NOT the same text:

- **Narration script** (what TTS reads): homophones disambiguated, symbols spelled out, numbers written as spoken digits, no code tokens. Multi-pronunciation characters and English identifiers inside Chinese text are the top failure sources — rewrite each into how it should be *said*.
- **Display text** (what the viewer reads): exact terms, code, punctuation preserved.

Write both, show the user the narration script, get a nod, then synthesize. Skipping the split produces mispronounced product names and comically read-out symbols.

## Step 4 — Synthesize incrementally

- Segment the narration script (per scene/step/cue), synthesize serially, **skip files that already exist** — an interrupted run resumes without re-burning quota.
- A voice change invalidates every cached file: pass the force flag and re-synthesize the whole track, and tell the user what that costs before running it.
- Report cost with numbers: chars × rate per run, and cumulative. Users forgive spend they were shown, not spend they discover.

## Provider quick reference

MiniMax:

```bash
mmx tts --voice <voice-id> --text "<segment>" --out seg-01.mp3   # per-segment; loop with skip-existing
```

OpenAI:

```bash
curl https://api.openai.com/v1/audio/speech -H "Authorization: Bearer $OPENAI_API_KEY" \
  -d '{"model":"tts-1","voice":"<voice>","input":"<segment>"}' --output seg-01.mp3
```

Other providers: each is one small shell script wrapping the provider CLI/HTTP API (input text → output mp3 per segment); the three commands any wrapper needs are audition, synthesize-segment, and skip-existing. Paste-in snippets for ElevenLabs / edge-tts / Azure / Google follow the same shape.
