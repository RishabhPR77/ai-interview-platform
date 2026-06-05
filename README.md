# 🎙️ AI Interview Platform

> **Adaptive AI-powered interview system with real-time multimodal behavioural analysis**

🥈 **1st Runner-Up — SSH '26 National Hackathon** &nbsp;|&nbsp; Private Repository · Active Development

---

## What It Does

Most interview tools just ask questions. This one *adapts* — it listens, watches, scores, and decides what to ask next based on how well you answered the last one.

Give it a job description and a resume. It generates a tailored question bank, conducts a full video interview, analyses your speech and body language in real time, and produces a detailed hiring report with a hire/no-hire recommendation.

---

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                   React / TypeScript UI                  │
│     Setup → Camera Check → Interview → Report           │
└──────────────────────┬──────────────────────────────────┘
                       │ REST (FormData / JSON)
┌──────────────────────▼──────────────────────────────────┐
│                  FastAPI Backend                         │
│                                                         │
│   /interview/start   →   Question Bank Generator        │
│   /interview/answer  →   Video Pipeline + Scoring       │
│   /interview/end     →   Final Report Assembler         │
└────────┬──────────────────────┬───────────────────────--┘
         │                      │
┌────────▼────────┐   ┌─────────▼──────────┐
│  Audio Analyzer │   │  Video Analyzer     │
│  (Whisper ASR)  │   │  (MediaPipe)        │
│                 │   │                     │
│  • Transcript   │   │  • Eye contact %    │
│  • WPM          │   │  • Gaze direction   │
│  • Filler words │   │  • Posture score    │
│  • Confidence   │   │  • Nervousness      │
│    language     │   │    indicators       │
│  • Pause count  │   │  • Gesture freq     │
└────────┬────────┘   └─────────┬───────────┘
         └──────────┬───────────┘
            ┌───────▼────────┐
            │  Groq API      │
            │  LLaMA-3.3-70B │
            │                │
            │  • Per-answer  │
            │    scoring     │
            │  • Adaptive    │
            │    next Q      │
            │  • Final report│
            └────────────────┘
```

---

## Key Features

### 🧠 Adaptive Question Engine
- Generates a role and resume-specific question bank upfront (more questions than needed)
- After each answer, LLaMA-3.3-70B selects the next question based on topic coverage gaps
- Weak answer → auto-generates a follow-up on the spot
- Suspiciously rehearsed answer → injects a curveball variant to test real understanding

### 🎥 Multimodal Behavioural Analysis
Every video answer is processed through two parallel pipelines:

| Signal | Source | What's Measured |
|---|---|---|
| Transcript | Whisper ASR | Full text of answer |
| Words per minute | Whisper | Speaking pace |
| Filler words | NLP | "uh", "um", "like", "you know" |
| Confidence language | NLP | Hedging vs. assertive phrasing |
| Eye contact % | MediaPipe iris landmarks | Gaze on-camera vs. away |
| Gaze direction | MediaPipe | Center / left / right / up / down |
| Posture score | MediaPipe Pose | Shoulder levelness, head-forward |
| Nervousness indicators | MediaPipe Face | Lip compression, brow furrow, hands near face |
| Gesture frequency | MediaPipe Hands | Hand movement rate per minute |

### 📊 Weighted Rubric Scoring
Interviews are scored across configurable dimensions that must sum to 100:
- `technical_depth`, `problem_solving`, `communication_clarity`, `confidence`, `cultural_fit`
- Each answer scored independently, final report aggregates with weights applied

### 🔁 Session Resume
If the candidate closes the browser mid-interview, the session is saved to localStorage and can be resumed exactly where it left off.

### 📄 PDF Report Export
Final report includes overall score, hire recommendation with confidence level, per-question breakdown, behavioral heatmap, strengths, weaknesses, and red flags — all exportable as PDF.

---

## Tech Stack

**Backend**
- `FastAPI` — stateless REST API, 3 endpoints
- `Groq API (LLaMA-3.3-70B)` — question generation, adaptive selection, scoring, final report
- `Whisper` — audio transcription and speech signal extraction
- `MediaPipe` — face mesh, pose, and hand landmark detection
- `OpenCV` — video frame extraction and preprocessing
- `Pydantic V2` — request/response validation

**Frontend**
- `React 18 + TypeScript` — full interview flow
- `Vite` — build tooling
- `pdfjs-dist` — resume PDF parsing on upload
- Session persistence via localStorage

---

## API

### Start an interview
```bash
POST /api/v1/interview/start
Content-Type: application/json

{
  "job_title": "ML Engineer",
  "job_description": "...",
  "candidate_name": "Rishabh Patidar",
  "candidate_resume": "...",
  "num_questions": 6,
  "interview_type": "mixed",
  "difficulty": "mid",
  "rubric": {
    "technical_depth": 35,
    "problem_solving": 25,
    "communication_clarity": 20,
    "confidence": 10,
    "cultural_fit": 10
  }
}
```

### Submit a video answer
```bash
POST /api/v1/interview/answer
Content-Type: multipart/form-data

session_id=abc-123
question_id=0
video=@answer.webm
```

### Get the final report
```bash
POST /api/v1/interview/end
Content-Type: multipart/form-data

session_id=abc-123
```

**Sample report response:**
```json
{
  "overall_score": 74.2,
  "recommendation": "hire",
  "recommendation_confidence": "medium",
  "strengths": ["Strong system design fundamentals", "Clear structured communication"],
  "weaknesses": ["Eye contact dropped on technical questions", "Hedging language on trade-off questions"],
  "red_flags": [],
  "breakdown": {
    "technical_depth": 7.8,
    "problem_solving": 6.5,
    "communication_clarity": 7.2,
    "confidence": 8.0,
    "cultural_fit": 5.5
  }
}
```

---

## How the Adaptive Logic Works

```
Generate question bank (1.5× num_questions asked)
         │
         ▼
   Ask question N
         │
         ▼
   Receive video answer
         │
   ┌─────▼──────┐
   │ Score answer│
   │ Audio + Video│
   │ + LLM content│
   └─────┬──────┘
         │
   ┌─────▼──────────────────────────┐
   │ Answer weak / vague?           │──► Generate follow-up on the spot
   │ Answer seems rehearsed?        │──► Inject curveball variant
   │ Topic coverage gap exists?     │──► Select question from that category
   │ All good?                      │──► Select next highest-priority question
   └────────────────────────────────┘
         │
   More questions remaining? ──► Loop
         │
         ▼
   Generate final report
```

---

## Project Status

🔒 **Repository is private** — the project is under active development by the team toward a production release.

Built at **SSH '26 National Hackathon** where it placed **1st Runner-Up** competing against top student teams nationwide.

---

## Team

Built by Rishabh Patidar and teammates at SSH '26.
