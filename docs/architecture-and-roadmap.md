# Pronunciation App — Architecture & Implementation Blueprint

Status: proposed architecture, not yet implemented.

## 1. System context

    Learner (web/mobile browser)
        |
        v
    Frontend: lessons, recorder, player, IPA feedback, progress
        |
        v
    Application Backend / API
       |             |              |
       v             v              v
    Assessment      TTS          Shadowing
    Adapter         Adapter      Module
       |             |              |
       v             v              v
    OpenPronounce   Kokoro       Captions / SRT / VTT
       |             |              |
       +-------------+--------------+
                     |
                     v
             Normalized DTOs
                     |
                     v
              Database / cache

Recommended first implementation: modular monolith with clear service boundaries. Split inference into separate services only when scaling, deployment or GPU isolation justifies it.

## 2. Core modules

### A. Recording and assessment

Responsibilities:
- Browser microphone permission and recording.
- Upload validation, format conversion and duration limits.
- Expected text and accent selection.
- Call OpenPronounce through a private adapter.
- Convert raw upstream response into a stable app-owned result schema.
- Return score, transcript, per-word phoneme feedback and prosody.

API proposal:
- POST /api/v1/assessments
- GET /api/v1/assessments/{assessmentId}
- DELETE /api/v1/assessments/{assessmentId}

Do not expose OpenPronounce directly to the public client. Add authentication, rate limiting, file validation and retention controls.

### B. TTS reference voice

Responsibilities:
- Map locale/accent to an approved Kokoro language code and voice ID.
- Generate reference audio.
- Cache by normalized text, voice, speed and model revision.
- Return audio metadata and a stable playback URL.

API proposal:
- POST /api/v1/tts
- GET /api/v1/tts/{audioId}

Store voice configuration centrally. Validate voice/language compatibility rather than accepting arbitrary IDs from clients.

### C. Shadowing / YouTube lesson

Responsibilities:
- Accept a YouTube URL as a lesson reference.
- Obtain transcript from a permitted source: authorized YouTube captions API access, creator-provided subtitles, or transcript uploaded/pasted by the user.
- Parse SRT/VTT and preserve start/end timestamps.
- Translate selected segments using a pluggable translation provider.
- Split into practice sentences while retaining original timing.
- Let learner replay one sentence, record a take and compare against target text.

API proposal:
- POST /api/v1/shadowing/sources
- POST /api/v1/shadowing/sources/{sourceId}/transcript
- GET /api/v1/shadowing/lessons/{lessonId}/segments

YouTube Data API notes:
- Caption track listing does not itself return caption text.
- Caption download is a separate operation and requires appropriate authorization; do not assume arbitrary public videos can be downloaded through the API.
- Do not build around scraping or bypassing access controls.
- Label machine-generated captions and translations; provide a correction path.

### D. Phoneme and alphabet lessons

Responsibilities:
- Present an accent-specific phoneme inventory.
- Play a sound sample and example word.
- Show IPA, spelling examples and mouth/tongue placement guidance.
- Track practiced sounds and mastery estimates.

A traditional British-English teaching chart commonly teaches 44 phonemes: 24 consonants, 12 monophthongs and 8 diphthongs. This is a teaching convention, not a universal inventory; General American differs in vowel inventory and realization. Choose a reference accent and have a qualified pronunciation educator validate every symbol, example and audio file.

Alphabet module:
- A–Z letter names (not phoneme sounds).
- Distinguish letter name from the sound(s) a letter can represent.
- Provide examples and audio; do not imply one letter always maps to one sound.

## 3. Application-owned data contracts

Assessment result proposal:
- id
- targetText
- locale (for example en-US)
- score
- transcript
- words[]: text, expectedIpa, heardIpa, confidence, feedbackCode
- prosody: pitch[], energy[]
- assessmentEngine: name, version

Shadowing segment proposal:
- id
- startMs, endMs
- sourceText
- translatedText
- translationStatus
- captionSource

These are application contracts, not copies of upstream response schemas.

## 4. Storage and privacy

- Store lesson content separately from personally identifiable user data.
- Obtain microphone permission and explain why audio is processed.
- Set a short default retention period for raw audio; allow deletion.
- Encrypt stored audio and restrict access by user/tenant.
- Do not use learner recordings for model training without explicit informed consent.
- Keep assessment summaries if useful, but avoid retaining raw audio indefinitely.
- Apply upload size/duration limits and malware/content checks.

## 5. Licensing and attribution gate

- OpenPronounce: upstream MIT license; preserve notice when redistributing code.
- Kokoro model: official model card declares Apache-2.0; retain license and notices for redistributed assets.
- Kokoro community voice packs: independently verify license, provenance, speaker consent and commercial-use permission.
- Vietnamese Storyvert asset: unverified; do not bundle, publish or use commercially until rights are established.
- Translation services, caption sources and datasets require separate terms review.

## 6. Suggested MVP sequence

### Phase 0 — technical spikes
- Install OpenPronounce locally and test its API.
- Generate US and UK reference samples with Kokoro.
- Measure startup, RAM/VRAM, CPU latency and audio quality.
- Confirm exact voice IDs and pinned revisions.
- Obtain the complete Vietnamese voice source and license evidence.

### Phase 1 — pronunciation MVP
- Record one sentence.
- Submit to assessment service.
- Show score, transcript and word-level expected/heard sounds.
- Replay learner recording and reference TTS.
- Save assessment summary and progress.

### Phase 2 — phoneme lessons
- Add validated accent-specific phoneme chart.
- Add sound samples, example words and mouth-position diagrams.
- Add A–Z letter-name lessons.
- Track per-phoneme practice.

### Phase 3 — shadowing
- Accept uploaded SRT/VTT or permitted transcript text.
- Segment transcript and preserve timestamps.
- Add translation provider abstraction.
- Replay one segment and record a learner take.
- Add progress and repetition controls.

### Phase 4 — production hardening
- Load testing and observability.
- Model worker scaling and audio cache.
- Security/privacy review.
- Human-rated pronunciation evaluation.
- Accessibility and mobile browser testing.

## 7. Acceptance criteria for first integrated demo

- Learner can record and submit a supported audio file.
- App shows assessment output without exposing internal model endpoints.
- Learner can listen to a US or UK reference voice from the official catalogue.
- Learner can replay a phrase and record a second attempt.
- UI labels automated scoring as approximate feedback.
- App imports a user-provided SRT/VTT and practices timestamped segments.
- No unverified voice pack or copyrighted model asset is redistributed.