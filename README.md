# Pronunciation App — AI English Speaking & Shadowing

An ELSA-inspired learning application concept combining phoneme-level pronunciation feedback, reference TTS voices, YouTube-based shadowing materials and interactive phoneme lessons.

> Repository status: documentation and technical blueprint. This is not yet a verified, runnable application. Feature descriptions define intended product scope and integration plans; they are not claims that the features are implemented.

## Product vision

Help English learners understand what they said, which sounds need work and how to practice again.

## Planned feature modules

| Module | Planned capability | Candidate technology |
|---|---|---|
| Pronunciation assessment | Record speech; score; transcript; word-level IPA feedback; prosody | https://github.com/Halleck45/OpenPronounce |
| Reference voice | US/UK reference audio for words and sentences | https://github.com/hexgrad/kokoro |
| Vietnamese TTS | Vietnamese explanation/reference voice, subject to rights and quality checks | Community Storyvert asset — unverified |
| YouTube shadowing | Captions/transcripts, translation, sentence segmentation and repeat practice | Authorized captions; user-provided SRT/VTT/transcript |
| Phoneme learning | Interactive English phoneme chart with sound samples and mouth-position guidance | Validated IPA lesson content + approved audio |
| Alphabet learning | A–Z letter names, examples and audio | Lesson content + TTS |

## Learner loop

1. Choose a word, sentence or video lesson.
2. Listen to the reference voice (US/UK).
3. Read the transcript and optional IPA.
4. Record the learner's voice.
5. Receive automated feedback and replay the relevant sound.
6. Practice the phrase again using shadowing.
7. Track progress without presenting automated scores as absolute truth.

## Suggested architecture

- Frontend: responsive web UI for lessons, recording, audio playback, transcript alignment and visual feedback.
- Application API: authentication, lesson management, progress, transcript processing and stable DTOs.
- Pronunciation service: isolated OpenPronounce runtime; do not expose internal model service publicly.
- TTS service: Kokoro runtime with US/UK voice mapping, audio cache and pre-generation.
- Media/transcript module: authorized caption retrieval or user-uploaded transcript parsing; sentence segmentation and translation provider abstraction.
- Storage: lesson metadata and progress; short-lived audio storage with retention/deletion rules.
- Background workers: model warmup, long audio jobs, batch TTS and media processing.

See docs/architecture-and-roadmap.md.

## Important cautions

- OpenPronounce is an experimental/self-hosted assessment building block. Upstream says English is calibrated; other supported languages are experimental.
- Kokoro generates reference audio; it does not evaluate learner speech.
- The official Kokoro model card declares Apache-2.0. Community voice packs need independent license and consent review.
- The mentioned Vietnamese Storyvert file is unverified because the supplied Drive URL is incomplete.
- YouTube transcript access is not unrestricted video downloading. Use authorized caption tracks, creator-provided files or user-provided transcripts. Respect API authorization, platform terms and copyright.
- A 44-phoneme chart is accent-dependent. Make the selected accent visible and do not mix US/UK IPA conventions without explanation.

## Related research repository

Technical research, source links, licensing notes and upstream limitations are maintained in https://github.com/Tranvanthuan1805/pronunciation-refs.

## Status

- [x] Initial technology research and source attribution documented.
- [x] Initial architecture and product scope documented.
- [ ] Verify Vietnamese voice pack and usage rights.
- [ ] Build proof of concept for OpenPronounce API integration.
- [ ] Build Kokoro US/UK voice proof of concept.
- [ ] Validate phoneme lesson content and audio.
- [ ] Implement authorized transcript ingestion and shadowing.
- [ ] Create evaluation dataset and measure feedback quality.
- [ ] Build end-to-end MVP and deployment pipeline.