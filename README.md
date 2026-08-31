# Interviewer Helper

Real-time response assistant for testing `ai-voice-interview-platform`.

## What is it

A tool that listens in real-time to questions from an interviewer (AI or human) during manual testing, generates a suggested answer with an LLM, and displays it on screen for the tester to read aloud — saving the tester from having to improvise technical responses while validating the platform's real voice pipeline.

## Why

`ai-voice-interview-platform` is a voice-based technical interview platform. Today, testing the complete flow requires a human to manually act as a candidate, which is slow and depends on the tester being able to improvise good answers. This project accelerates those manual tests.

## How it works (architecture)

Low-latency pipeline:

1. **Audio capture**: loopback from system audio output (what plays through speakers/headset)
2. **VAD**: detects when the interviewer finishes speaking → triggers transcription close
3. **STT (streaming)**: transcribes the question in real-time (low latency)
4. **LLM (streaming)**: generates suggested answer (technical, AI/Data Eng context)
5. **Console UI**: displays question + answer streaming
6. **Metrics**: latency between end-of-question and suggested response appearance

Stack: Python, cloud STT/LLM providers with local option (pluggable).

## Roadmap (Milestones)

- **M1**: Audio loopback capture (PoC) — Windows
- **M2**: Real-time STT — transcription to console
- **M3**: VAD / question segmentation
- **M4**: LLM + suggested response
- **M5**: Latency measurement and tuning
- **M6**: Console UI polish
- **Future**: Spanish/bilingual support, human mic capture for auto-detection, 100% automation via API, cloud portability

## Testing environments

1. **Against platform**: real session against `ai-voice-interview-platform` running locally (Clara, 8 fixed questions)
2. **Call with colleague**: Zoom/Meet session with engineer acting as interviewer (more variety, controlled environment)

## Prerequisites

- Python 3.10+
- `ai-voice-interview-platform` running locally (`docker-compose up`)
- API keys for chosen STT and LLM provider
- Audio device with accessible loopback output (headset/speakers)

## Development

```bash
# Clone / enter the project
cd "C:\Users\ASUS\CodingProjects\interviewer helper"

# (To be implemented) Create virtual env and install deps
python -m venv venv
source venv/Scripts/activate  # Windows
pip install -r requirements.txt

# (To be implemented) Run
python main.py
```

## Status

🟡 **In planning** — initial task list to be defined. Milestone 1 (audio capture) in progress.
