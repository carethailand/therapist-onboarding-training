# Therapist Training & Exam Platform — Spec & Plan (Draft v1)

*Prepared June 2026. This is a working draft to think through what we want, what's feasible on GitHub Pages today, and the path to the secured ABAT version later.*

---

## 1. Purpose

A web platform where a **new therapist (trainee)** can:

1. Log in with a **traceable ID** assigned by us.
2. Work through **self-paced coursework** — links to our Canva learning resources.
3. Sit a **timed exam** (2-hour limit) with mixed question types, shown in random order.
4. Submit, and see only a **"Thank you — we'll be in touch"** message.

We (the **trainer**) receive the trainee's **ID, score, pass/fail, and time used** — the trainee never sees their own score on screen.

Phase 2 reuses the same engine for our **ABAT coursework**.

---

## 2. What's built in this draft

A working static prototype you can drop into your GitHub repo (the same one as the CARE binder):

| File | What it is |
|------|-----------|
| `index.html` | The whole platform — login, course links, exam, timer, scoring, thank-you screen. One self-contained file. |
| `questions.json` | The exam content + answer key. **Already loaded with your real 61-question exam** (62 marks, pass 53). |
| `trainees.json` | The roster of admin-assigned trainee IDs and PINs. |
| `PLATFORM_SPEC.md` | This document. |
| `RESULTS_SHEET_SETUP.md` | Step-by-step guide to wire results into a Google Sheet. |
| `Onboarding-Manual-2026.pdf` | Your training manual, downloadable from the dashboard. |

To preview: put the four files in a GitHub repo, turn on **GitHub Pages**, open the URL. Log in with `RBT-2026-001` / PIN `1234`.

> **Local testing note:** opening `index.html` directly from your desktop (file://) will fail to load the JSON files because browsers block local file reads. It works fine once it's on GitHub Pages, or if you run a tiny local server (`python3 -m http.server` in the folder, then visit `localhost:8000`).

---

## 3. Feature detail

### 3.1 Trainee ID (traceable, admin-assigned)
- You add each trainee to `trainees.json` before they start, e.g. `RBT-2026-001`.
- The ID is what flows into every score record, so every result is traceable to one person.
- Optional 4-digit PIN per trainee so someone can't just guess another person's ID.

### 3.2 Self-paced coursework
- A **"Download Training Manual (PDF)"** button at the top of the dashboard serves `Onboarding-Manual-2026.pdf` from the repo. Replace that file (keep the name, or update `MANUAL_FILE` in `index.html`) to swap manuals.
- Below it, a list of modules, each with an **"Open in Canva"** button.
- Edit the `COURSES` list near the top of `index.html` to paste your real Canva share links.

### 3.3 Exam engine
- **2-hour countdown timer**, sticky at the top. Turns amber at 10 minutes, red at 2 minutes, and **auto-submits at zero**.
- **Random order** every attempt — questions shuffle, and answer options within each question shuffle too.
- **Question types supported:**
  - Multiple choice (your 60 questions)
  - Select-all-that-apply (your Q61)
  - Matching (term → definition) — built and ready for future exams
  - Drag-the-word-into-the-blank — built and ready for future exams
- **Auto-scoring** against the answer key. Multiple-choice and select-all are all-or-nothing; matching and drag-fill award partial marks per correct slot.
- **Time used** is recorded in minutes.

### 3.4 What the trainee sees vs. what you see
- **Trainee:** after submitting → *"Thank you for submitting. We will get in touch with you regarding your result."* No score shown.
- **Trainer:** a hidden result panel unlocks with a trainer code (`trainer2026`, change it in `index.html`) showing score, pass/fail, % , and time used.

---

## 4. The honest limitation: getting results back to you

This is the one real constraint, and it's worth understanding clearly.

**GitHub Pages serves files only — there is no server.** It can show the exam and grade it in the trainee's browser, but it has nowhere to *store* a submitted result. So the score lives in the trainee's browser unless we actively send it somewhere. There are three realistic ways to do that:

| Option | How it works | Cost | Effort | Good for |
|--------|--------------|------|--------|----------|
| **A. Google Form / Sheet** (recommended) | The exam silently posts ID + score + time to a Google Form you own. Results pile up in a Google Sheet. | Free | ~30 min setup | A clean running log of every submission, sortable, no inbox clutter. |
| **B. Form-to-email (Formspree etc.)** | Each submission emails you the result. | Free tier | ~10 min | Low volume; you don't mind one email per trainee. |
| **C. Real backend later** | A small server (or Google Apps Script) receives, stores, and also keeps the answer key off the trainee's device. | Low–moderate | Developer needed | The actual ABAT exam where stakes are high. |

The prototype has a single line — `SUBMIT_ENDPOINT` — where you paste a Form/Formspree URL when you're ready to go live. Until then it just logs the result to the browser console and the trainer panel.

### The second limitation: the answer key is in the files
Because grading happens in the browser, the answer key sits inside `questions.json`. A technically savvy trainee could open it and read the answers. For **self-paced practice this is acceptable.** For the **real ABAT certification exam it is not** — that's exactly what Option C above fixes (the key and grading move to a server the trainee can't see). I've flagged this so it's a deliberate decision, not a surprise.

**Recommendation:** ship this now as the *practice / internal-training* exam with Option A (Google Sheet). Plan the server-side version (Option C) for the graded ABAT exam.

---

## 5. How to hand me exam content (question-sheet format)

You already did it perfectly — a doc with the questions and a key on the last page. Going forward, any of these works:

- A Word doc / Google Doc with questions then an answer key (what you sent), **or**
- Filling in the `questions.json` pattern directly.

The four shapes the engine understands:

```jsonc
// Multiple choice — "answer" is the index of the correct option (0=first)
{ "id":"q1", "type":"mcq", "prompt":"...?",
  "options":["A","B","C","D"], "answer":1, "points":1 }

// Select all that apply — "answer" is a list of correct indexes
{ "id":"q61", "type":"multiselect", "prompt":"... select all",
  "options":["Full Physical","Gestural","Textual"], "answer":[1,2], "points":2 }

// Matching — "answer"[i] = which right-side item matches left-side item i
{ "id":"q3", "type":"matching", "prompt":"Match each term.",
  "left":["Prompt","Shaping"],
  "right":["Reinforcing approximations","A cue that helps"],
  "answer":[1,0], "points":2 }

// Drag word into blank — one word per blank, in order
{ "id":"q4", "type":"draganddrop", "prompt":"Fill the blanks.",
  "textParts":["An FBA identifies the "," of a behavior."],
  "wordBank":["function","duration"], "answer":["function"], "points":1 }
```

Top of the file sets the rules: `"timeLimitMinutes":120, "totalMarks":62, "passMark":53`.

---

## 6. Roadmap

**Phase 1 — now (this draft).** Internal training exam on GitHub Pages. Wire up a Google Sheet for results. Use it with a few trainees, gather feedback.

**Phase 2 — harden for ABAT.** Move grading + answer key to a server (Option C) so the exam is tamper-proof. Add: one-attempt-per-trainee enforcement, proctor/login audit log, optional question banks (randomly draw 60 from a pool of 100 so no two trainees get the same paper).

**Phase 3 — nice-to-haves.** Results dashboard for trainers, automatic pass/fail emails, per-question analytics (which questions trip people up), certificate generation on pass.

---

## 7. Decisions locked in
- **Results → Google Sheet** (Option A). Setup steps in `RESULTS_SHEET_SETUP.md`.
- **One attempt per trainee.** After submitting, that trainee ID is blocked from re-taking on that device (stored in the browser). An admin can reset it by clearing the browser's site data, or fully enforce it server-side in Phase 2.
- **ABAT:** always the full question set (no random subset).
- **No trainee-facing email** — admin handles results contact.

## 7b. Other defaults (change any of these)
- Trainer code is `trainer2026` and demo PINs are in `trainees.json` — change before real use.
- Matching/drag-fill are all-or-partial scored; your current exam doesn't use them yet but they're ready.
- "Thank you" wording is editable in `index.html`.

## 8. Note on retake blocking
The one-attempt block currently lives in the trainee's browser, so it holds on the same computer but a determined trainee could bypass it (different browser/device, or clearing data). For the internal practice exam that's acceptable. For ABAT, the Phase-2 server enforces it properly — the result is checked against the trainee ID centrally, not on their machine.
