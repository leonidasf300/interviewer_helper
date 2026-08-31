# Interviewer Helper — Development Plan

Real-time response assistant for testing `ai-voice-interview-platform`.

## Context

`ai-voice-interview-platform` (sibling repo at `C:\Users\ASUS\CodingProjects\ai-voice-interview-platform`) is a web-based platform (React+Vite / FastAPI / Postgres+Redis) where an AI interviewer ("Clara") conducts technical interviews by voice. Today, testing the complete flow requires a human to manually act as a candidate — speaking real technical answers to the browser — which is slow and depends on the tester being able to improvise good answers.

This project (`interviewer-helper`) is a **standalone, decoupled tool** (separate repo, no shared code with the platform) that assists a **human tester** during those manual tests: it listens in real-time to questions asked by the AI interviewer (via actual system audio), generates a suggested answer with an LLM, and displays it on screen for the human to read aloud into their own microphone. The human is still the one "answering" — the tool just saves them from having to think/improvise the technical response, and because it's real human voice (not synthetic), it also helps validate the platform's real voice pipeline (Clara's TTS + browser's Web Speech Recognition).

It is not a 100% automated candidate nor intended for real job interviews — it's internal QA tooling to support manual testing. Latency matters because the tester needs the suggested text quickly to read it naturally before the platform marks timeout or silence.

V1 runs locally (same PC or separate PC on same network); the design should allow moving it to another PC/cloud later without major redesign.

Since the pipeline listens to actual system audio (it doesn't depend on the platform's API), it is **agnostic to who asks the question** — it works the same whether the question comes from Clara (synthetic voice) or a real person in a call (Zoom/Meet). This is leveraged to expand testing: Clara only has 8 fixed questions, so in addition to testing against the platform, test sessions will be held in calls with a colleague engineer acting as interviewer (both aware it's a tool test/practice, not a real interview) — this provides more question variety and a more controlled environment to validate the complete pipeline.

## Decisions already made

- **Single mode**: human-in-the-loop assistant (100% automated candidate via API is ruled out for now — can be revisited as a separate CI/regression tool in the future).
- **Question source: actual system audio (loopback) + STT**, not the platform's API — prioritizes full fidelity (how a real candidate would experience it) and keeps the tool reusable against other interview platforms in the future, not just this one.
- **GPU**: not confirmed (to be determined/detected at setup) — affects whether local Whisper/LLM are viable for minimum latency.
- **STT/LLM providers**: hybrid prioritizing latency — use cloud streaming APIs where they give lowest latency (e.g., Deepgram/AssemblyAI-style STT, Anthropic/OpenAI-style LLM with streaming), with pluggable architecture to swap providers or use local (Whisper/Ollama) without redesign.
- **Stack**: Python — best support for Windows audio loopback capture and integration with AI providers.
- **MVP language**: English first (reduces language detection complexity / bilingual prompts); Spanish added after pipeline is validated.
- **Target role**: AI Engineering / Data interviews, open to other roles later.

## Overall objective

Build an assistant that listens in real-time to questions from an AI interviewer during manual testing of `ai-voice-interview-platform`, transcribes the question, generates a technical suggested answer with an LLM, and displays it on screen as fast as possible for the human tester to read aloud — minimizing latency between question-end and suggested-response appearance.

## Architecture (pipeline)

1. **Audio capture**: loopback from system audio output (what plays through speakers/headset) to hear Clara's question. The tester's mic is not needed for core pipeline (the human speaks directly to the platform), but remains available for future improvements (e.g., detecting when human starts responding to know the current question has closed and reset state).
2. **VAD (speech-end detection)**: determines when Clara finishes speaking and triggers transcription close — critical to avoid waiting too long or cutting the question short.
3. **STT (transcription)**: streaming, low-latency — so by the time silence is detected, the final transcription is (nearly) ready.
4. **LLM**: generates suggested answer streaming (so tester can start reading the first tokens while the rest are still generating), with a prompt oriented to technical AI/Data Eng interviews.
5. **Console UI**: displays transcribed question and streaming suggested answer, simple and readable in real-time.
6. **Latency instrumentation**: measures and displays time between question-end and first response tokens, to tune each pipeline stage.

Specific library decisions (exact STT/LLM provider, VAD library, audio loopback library) remain **open** — decided during each milestone's implementation phase, not fixed in this plan.

## Roadmap

1. **M1 — Audio capture (PoC)**: capture system audio output (loopback) on Windows and confirm Clara's voice is heard correctly during real platform test session.
2. **M2 — Real-time STT**: live transcription of captured audio, displayed to console; validate accuracy and latency (English only).
3. **M3 — VAD / question segmentation**: detect end of question to know when to close transcription and trigger next step.
4. **M4 — LLM + suggested response**: send transcribed question (with AI/Data Eng interview context) to LLM and display response streaming in console.
5. **M5 — Latency measurement and tuning**: instrument timings per stage, show end-to-end metric, adjust to minimize delay from question-end to first suggested-response word.
6. **M6 — Console UI polish**: live view with transcribed question, streaming response, and session history.
7. **Future (outside this plan)**: Spanish/bilingual support, human mic capture for auto-detecting when response starts, 100% automation via API (for CI/regression, no human), cloud portability, human practice module (AI as interviewer, human practices with post-interview feedback) as separate project/module.

## Verification

Two testing environments, since Clara only has 8 fixed questions:

1. **Against platform**: real session against `ai-voice-interview-platform` running locally (`docker-compose up`), listening to Clara's questions.
2. **Call with colleague**: Zoom/Meet session with colleague engineer acting as interviewer (both aware it's a tool test/practice) — provides more question variety and controlled environment to validate pipeline outside Clara's fixed set.

Checks per milestone:
- M1: recording from each environment, confirm captured audio matches interviewer voice (Clara or colleague).
- M2: live transcription checked manually against what was actually said, at least 3 questions from each environment.
- M3: question-end detection correct in repeated tests (no premature cuts, no excessive waits), both environments.
- M4: suggested response coherent and technically sound for typical AI/Data Eng questions, both environments.
- M5: end-to-end latency report (question-end → first response tokens) across at least 3 runs per environment.

## Prerequisites / external dependencies

- `ai-voice-interview-platform` running locally (`docker-compose up`) for end-to-end testing.
- API key(s) for chosen STT and LLM provider(s).
- Audio device with accessible loopback output (headset/speakers from test PC).
