## Multiple paused quizzes + bigger logo (2026-11-10 — MultiPause)

### Bug fixed: only the last paused quiz was kept
Resume state used to live under the single key `allSubjectsResume`, so pausing
a second quiz overwrote the first. Resume storage is now a LIST under
`allSubjectsResumes`: every paused quiz is kept (upserted by quiz key, so
pausing the same quiz twice never duplicates it). Home shows an
"⏸️ ఆగిన క్విజ్‌లు" section listing EVERY paused quiz — title, subject,
answered/total count, remaining time (exam) or mode, saved date/time — each
with its own "▶️ కొనసాగించు" (resume exactly that quiz: same question,
answers, marked, remaining time) and "❌ తొలగించు" (discard only that one)
buttons. Resuming removes that entry from the list (it is live again);
submitting removes only its own entry. The old single
"▶️ ఆగిన క్విజ్ కొనసాగించు" button is gone. One-time migration: an old
single `allSubjectsResume` entry is moved into the list on load and the old
key is deleted. Mixed/Quick/Daily specials still cannot be saved for later.
Paused quizzes live only on the device — clearing browser data loses them.

### Logo — bigger header + gold letters
- Header logo is now **56×56** (was 36×36), circular crop kept; header CSS
  adjusted so it fits on desktop and phone.
- The user's logo letters (and circle ring) were recolored from silver/white
  to **gold** on black; `icon-512.png`, `icon-192.png`, `favicon.png` were
  regenerated from the gold version (header, favicon, home-screen icon and
  manifest all use the gold logo now).

### Verification (jsdom, `work/multi-pause-fix-2026-11-10/test-multi-pause.js` + `test-supplement.js`)
49/49 + 7/7 checks pass with `window.confirm` stubbed to THROW: pause quiz A
(3 answers) then quiz B (2 answers) → home lists BOTH with correct counts;
resume A → A's answers + remaining time + idx restored, B still listed;
re-pause A → no duplicate; resume B → B restored; discard → only that entry
gone; submit a resumed quiz → only its entry removed; legacy single resume
migrates into the list and the old key is deleted; pause freeze/resume,
language toggle, explanation bold with no `**` leak (s01/s18/s19 samples),
revision Day 1 → Day 4 advance, back navigation, submit modal with no native
confirm, totals 7940/7835/105/205/368, 22 subjects, footer/source 0 matches,
header logo 56×56, favicon/manifest links, both inline scripts pass
`node --check`. Totals unchanged; questions.js / explanations.js / concepts.js
untouched.

## Submit fix + Pause (2026-11-10 — SubmitPauseFix)

### Submit bug — root cause (proven in jsdom before the fix)
`Submit` used the browser's native `confirm()` dialog. In sandboxed iframes and
mobile WebViews `confirm()` returns `undefined` (falsy), so the click silently
did nothing — no result, no saved score — while last-question `Next` (which
called `submitQuiz()` directly) still worked. Reproduced: 9/9 checks in
`work/submit-pause-fix-2026-11-10/repro-submit-confirm.js`.

### Fix — in-app confirm modal, zero native confirms left
- Submit now opens an in-app modal ("క్విజ్ సబ్మిట్ చేయాలా?") with live
  counts (answered / skipped / marked-for-review) and two big buttons:
  "✅ సబ్మిట్ చేయి" (submits) and "❌ ఇంకా రాస్తాను" (closes, quiz continues,
  timer keeps running). Drawer Submit opens the same modal.
- ALL native `confirm()` calls replaced (grep `confirm(` in `index.html` = 0):
  submit — old line 768; clear-results-history — old line 358; reset-all-progress
  — old line 439. All three now use the same in-app modal pattern.
- Timer fix included: starting / resuming a quiz no longer eats 1 second
  (the old code decremented `remaining` immediately on start). Resume also
  restores the exact remaining time and does not count paused/away time in
  the result's elapsed time.

### Pause — exam mode AND practice mode (not revision mode)
- New "⏸️ పాజ్" button in the fixed quiz menu bar and in the drawer (☰ Menu).
- Pausing stops the timer immediately (interval cleared, remaining frozen),
  saves the full resume state, and shows a phone-friendly overlay with the
  remaining time and answered count, plus three big buttons:
  1. "▶️ కొనసాగించు" — resume in place, timer restarts from the SAME remaining time.
  2. "💾 సేవ్ చేసి తర్వాత రాస్తాను" — save + exit to the subject page; home
     "▶️ ఆగిన క్విజ్ కొనసాగించు" restores exactly the same question, answers
     and remaining time. (Only for chapter-set quizzes; Mixed/Quick/Daily
     specials cannot be saved for later, so that button is hidden there.)
  3. "❌ క్విజ్ వదిలేయి" — discards the saved resume and goes to the subject page.
- While paused the timer never ticks and answers can never be lost or submitted.
  Phone back button while the pause overlay is open only closes the overlay
  back to the paused quiz (timer stays frozen) — it never discards the quiz.
  The menu-bar button then reads "▶️ కొనసాగించు" to resume.

### Logo — user-supplied image (final)
- The user sent his own logo image (silver serif MR in a thin circle on black)
  and asked to use it for EVERYTHING. It is now the logo everywhere: header
  (circular, next to the title), `favicon.png`, home-screen icons
  `icon-192.png` / `icon-512.png`, `apple-touch-icon`, and `manifest.json`
  (theme/background black to match). The earlier 3D/glossy SVG logo was replaced.

### Verification (jsdom, `work/submit-pause-fix-2026-11-10/test-submit-pause.js`)
80/80 checks pass with `window.confirm` stubbed to THROW (so every passing
action also proves no native confirm ran): submit modal open/cancel/confirm,
result saved + wrong answer in revision queue, drawer submit, last-Q Next,
pause freeze/resume/save-exit/restore/discard in exam mode, pause in practice
mode, back-button-closes-overlay, clear-results + reset modals, and regressions
(totals 7940/7835/105/205/368, 22 subjects, back navigation, revision
Day 1→4→7→30, results view, logo/manifest links, language toggle, footer/source
0 matches, explanations with bold and no `**` leak in s01/s18/s19 samples,
both inline scripts pass `node --check`). Totals unchanged; questions.js /
explanations.js / concepts.js untouched.

### Upload
Same 10 files as before (`index.html`, `questions.js`, `concepts.js`,
`explanations.js`, `README.md`, `.nojekyll`, `favicon.png`, `icon-192.png`,
`icon-512.png`, `manifest.json`). **Upload via GitHub Desktop or `git push`** —
`explanations.js` is ~32 MB, over GitHub web upload's 25 MB per-file limit.


## Added 2026-11-04 — 7 new PDFs, subject-wise and topic-wise (appended, nothing existing changed)

1,102 genuinely new questions appended at the END of existing chapters (existing questions, chapter ids, positions and explanation keys unchanged, so saved progress/bookmarks/wrong lists carry over). New totals: **6,966 questions (6,862 scored + 104 open), 205 chapters, 336 sets** (was 5,864 / 5,778 / 86 / 305).

Per-file inventory (parsed/transcribed → finally added):
- Geography_All_Questions.pdf: 121 parsed → 2 added.
- Geography_Questions_All_Papers.pdf: 242 parsed, no key in file → 91 added (106 in first pass, minus 15 SI Mains duplicate-pair drops).
- Geography_Questions_All_Papers__2.pdf: 220 parsed (190 + 30 appendix) → 11 added.
- Geography_Questions_Combined.pdf: 56 parsed (44 text + 12 image-transcribed AP Constable 2018 Telugu pages) → 12 added.
- Geography_Questions_Compiled.pdf: 170 parsed (148 + 22) → 101 added.
- DOC Sandeep Geography book, 48 scanned pages: 452 transcribed (India book pp. 124–153 Q1–307 + Telangana book pp. 345–362 Q1–145) → 382 added.
- CamScanner General Science + District Court paper, 54 scanned pages, byte-identical to the earlier Science-app file (545 reused, not re-transcribed) → 503 added.
Cross-file duplicates removed: 45 in the first dedup pass, plus 16 SI Mains 2016 duplicate pairs (All_Papers truncated copy vs Compiled complete copy, kept once, complete version) — total new after all dedup: geo 217, DOC 382, CamScanner 503 = 1,102. Excluded: 12 Compiled AEE answers-only items with no option list in any file (cannot be MCQs); 17 no-option items in the first pass (including 2 All_Papers AEE items already in the app, matched by source+Q and verified by stem).
Answer sources for the 1,102 new questions: printed key 880 (DOC 378 + CamScanner 501 + geo 1), official-searched 48 (Group-IV 2018, SI Prelims 2016, SI Mains 2016 SET-A), searched-verified 156, open 18. New open questions (unscored, reason shown per question): geo 12 (2 official-deleted, 2 with no correct option, 2 with missing lists, others with conflicting/unverifiable figures), DOC 4 (key digit finger-obscured in scan), CamScanner 2 (options cut off in scan, as before).
Biggest chapter growth: Telangana Geography s03-c50 30 → 196 (+166), Geography Other s04-c12 6 → 86 (+80, forests/parks/misc items with no dedicated chapter), Geography Physical Features 74 → 137, Rivers 44 → 105, Climate 41 → 90, Minerals & Industries 35 → 88. Full before→after table is in work/add/merge_report.json. No new chapters were created; no paper name appears in any chapter title.
New explanations: 1,084 (geo 205 + DOC 378 + CamScanner 501 reused), mean 101.6 words, bold 2–8 phrases each, 0 unmatched **, 0 missing/extra. New questions have Telugu (default) + English toggle versions like the rest of the app (geo English = original source English, DOC/CamScanner English = translated from Telugu, answer order unchanged).
Key disagreements NOT changed: existing app answers were never changed, even where a new file's key or a searched answer differs — see work/add/ for the flagged cases (e.g. SI Mains 2016 Q65 Korba official key maps to MP, Q274 KAPP-3 printed key 800 MW vs 700 MW elsewhere, TG plateau Q9, Yamuna Q89, Simlipal matching) — check against the book before relying on those.
Home footer and per-question source lines remain removed (0 matches in index.html, preview and this zip), as the user requested.

# All Subjects Quiz — GitHub Pages (Free, ₹0)

Telugu-medium quiz app for all subjects: **5,864 questions, 17 subjects, 205 syllabus chapters, 305 sets of 30**.

**Browsing is syllabus topic-wise, NOT paper-wise.** After choosing a subject, you see that subject's syllabus chapters (e.g. Polity → ప్రాథమిక హక్కులు), with questions from ALL papers grouped under the chapter they belong to. A paper name appears ONLY as the source label inside each question panel, never as a browsing topic. The previous paper-wise version (405 topics, 433 sets) is superseded by this version; question text, options, answers, answer sources and explanations are unchanged — only the grouping changed.
Every question has a deep Telugu explanation (scored questions only); important phrases in the
explanation are bolded — nothing else is auto-bolded.

## The key feature: subject-first flow

1. **Step 1 — "Ee subject rayali anukuntunnavu?"** — Home shows ONLY a subject grid with
   per-subject total / scored / open / chapters / sets. Until you tap a subject, NO chapter
   list is shown at all (this was fixed after the user reported the old version dumping the
   whole syllabus on the home screen).
2. Tap a subject → **only that subject's syllabus chapters, chapter-wise, and its sets of 30**
   are shown below. Search also works within the chosen subject only.
3. **"All subjects — Mixed"** card is a mixed-quiz mode only — it shows NO chapter list.
   Mixed / quick / daily quizzes then draw from everything; when a subject is chosen, those
   specials draw from that subject only, and wrong/bookmark revision is also filtered to it.
4. Tap a chapter → tap a set (30 questions) → choose Practice or Exam mode.
5. **Order inside each chapter:** Sandeep book questions first, then AP papers, then TS papers,
   each in original order — so a chapter reads coherently.
6. **Progress note:** chapter ids changed in this regrouping, so set-progress, bookmarks and
   wrong-question lists saved under the old paper-wise topic ids do NOT carry over; the app
   simply starts fresh under the new chapter ids (same storage keys, no crash, old entries
   are ignored).


## Second addition — Economy & Telangana Schemes books (2026-11-04, AddedEconomy)

Two new scanned Sandeep books, transcribed from page images, deduplicated against the app, appended to existing chapters only (no existing question/answer/chapter/id/position/explanation changed; no new chapters):

- DOC-20261104-WA0001 (103 scanned pp., Sandeep Indian Economy book pp.160–222 + Telangana Economy section pp.363–399): transcribed 724 entries (DOC A 190 + DOC B 196 incl. 16 duplicate-scan entries removed + DOC C 192 + DOC D 166) → after dedup, Economy 420 scored + Telangana 264 scored new. One coordinator correction: Topic 6 Q10 (book p.176) stem/options were read from the scan (the first worker had attached Q8's options); key 1 (B and D only) retained.
- CamScanner_04-11-2026_11.29 (23 scanned pp., Sandeep Telangana Schemes & Laws book pp.400–421, some pages sideways): transcribed 157 candidates (Q1–84 + Q85–157) → after dedup, 140 scored + 1 open new. PDF p.4 of the Economy file and PDF pp.36/46/99 duplicates were counted once.
- Dropped, not added: Economy Topic 2 Q34 (only 3 options visible, option 4 cut at book p.166) — the complete scored version already exists in the app (s06-c01#39, Service sector), so the incomplete copy was dropped rather than kept as a second open question.
- Open new question (1): Cam Q79, Telangana Harita Nidhi launch year — only options 2019/2020 visible (cut at book p.410; PDF p.13 top verified to be the Q85 tail, not Q79 options); printed key 3 known but option 3 text unknown, so unscored.
- New totals: 7,791 questions (7,686 scored + 105 open), 205 chapters, 363 sets (was 6,966 / 6,862 / 104 / 336). Changed chapters (13): s03-c50 196→225, s03-c51 66→401, s03-c52 10→51, s04-c08 56→135, s06-c01 49→119, s06-c02 41→55, s06-c03 24→53, s06-c04 17→64, s06-c05 46→118, s06-c06 18→36, s06-c07 43→52, s06-c08 25→27, s06-c10 19→99. New answers: printed key 824, open 1. New explanations: 824, all 80–150 words, 2–8 bold phrases, 0 unmatched **. English toggle versions: Economy batch English was fully rewritten after the first draft was found to use Latin-script transliteration (gibberish); the published English was spot-checked clean. Cross-batch duplicates: 0. Footer and per-question source lines remain absent from the app UI.
- Key doubts NOT changed (printed keys followed as given): Cam Q3 (Mahalakshmi Jan 2024 vs Dec 2023 free bus start), Q12 (Cheyutha key for a ₹10 lakh Aarogyasri question), Q29 (scheme chronology order), Q36 (TASK date), Q46 (tank restoration statement marked wrong), Q104 (options cover only statements 1–2), Q106 (MeeSeva 2.0 vs Bhu Bharati as Dharani replacement), Q107 (Breastfeeding Week statement marked neither true), Q93 (Arogya Lakshmi date vs app s03-c51#12). Also noted, not changed: two TG candidates duplicate app OPEN questions (s06-c09 n29, s03-c50 n59) — their Sandeep printed keys exist, but existing app questions were left untouched per the no-change rule; the user may later decide to score those two.

## Neat questions (2026-11-04 — all stems reformatted, unreadable stems rewritten)

**What "neat" means in this build.** Two parts, applied to all 7,791 questions:
1. **Formatting (all questions):** a line break is inserted before each statement/list marker that was running inline mid-stem — ప్రకటన-1/2, ఎ./బి./సి./డి., A./B./C./D., a)/b)/c)/d), 1./2./3./4., I./II./III., జాబితా I/II, List I/II, and a trailing "సరైన … ఎంచుకోండి/ఎంచుకొనుము" instruction on its own final line — in the Telugu stem and, in parallel, in the English stem (se). No wording, number, name, option, or answer was changed by formatting; only a space before a marker became a newline. Stems that already had line breaks (212) were kept as-is except where a marker was still inline.
2. **Rewriting (unreadable subset only, 427 stems):** stems flagged for Latin connector verbs (sitting, containing, such that…), glossary Telugu-English mix, Telugu-script transliteration of whole English phrases (e.g. "అస్ పెర్ సొకిఒ ఎకొనొమిక్ ఔత్లూక్"), or a >300-char unbroken paragraph were rewritten in clear, simple, natural Telugu from their clean English original (se) where present, preserving all facts, numbers, names, and question intent. Stems that were already clean Telugu were NOT rewritten — formatting alone is their fix.
**Counts:** total questions unchanged 7,791; Telugu stems gaining ≥1 newline: 1,996 (plus 60 already-neat stems gaining further breaks); English stems gaining ≥1 newline: 2,205; already-neat and unchanged: 152; rewritten: 427 (Current Affairs & GK 201, Reasoning & Quantitative 141, Telangana Specific 38, History 15, Ethics/Society 10, Geography 6, Economy 6, Polity 5, Environment 2, Biology 2, General Science 1); left unchanged as ambiguous: 0 — every flagged stem had a clean English original to translate from. The full before/after list is in work/neat/rewritten_log.json.
**UI:** #qStem keeps white-space:pre-wrap with textContent rendering (no markup leak), line-height raised to 1.8 with a little more bottom margin so multi-line stems breathe. Language toggle behaviour unchanged.
**Known limitation (not hidden):** options were NOT rewritten in this pass, so a small number of Reasoning / Current Affairs options still carry the old glossary mix (e.g. "మాత్రమే B is correct"). Stems are clean, so the question intent is clear; options can be cleaned in a follow-up pass if the user asks.
**Examples:** statement — s01-c01#4 now shows "పరిగణించండి:" then "ఎ. …" then "బి. …" on separate lines; matching — lists split at జాబితా I/II; rewritten garbled — s12-c08#10 "Eight ప్రజలు అనేవి sitting…" → "ఎనిమిది మంది వ్యక్తులు నలుగురు చొప్పున రెండు సమాంతర వరుసలలో కూర్చున్నారు…"; rewritten transliteration — s03-c51#22 "అస్ పెర్ సొకిఒ ఎకొనొమిక్ ఔత్లూక్ 2023…" → "తెలంగాణ ప్రభుత్వ సామాజిక-ఆర్థిక ముఖచిత్రం (Socio Economic Outlook) 2023 ప్రకారం…".

## Language toggle (2026-11-04 — per-question Telugu / English, default Telugu)

Above every question in the quiz view there is a small **"🌐 Choose language: తెలుగు | English"** control.
- **Default is Telugu on every new question.** Tap "English" on that question and ONLY its stem + options switch to English; tap "తెలుగు" and it switches back instantly.
- **Answer index, scoring, progress, bookmarks, palette, timer and the explanation never change** when you switch language — the selected answer is language-independent, and the explanation (including its correct-answer line) stays in Telugu even in English mode.
- **Reset rule:** moving to a different question (Next / Prev / palette jump) resets the toggle to Telugu. Re-rendering the same question after answering (e.g. showing the practice-mode explanation) does NOT reset it.
- **Shuffle-safe:** with question/option shuffle ON, the English options are shuffled in exactly the same order as the Telugu options, so the answer mapping cannot break. Resumed quizzes also restore the shuffled English options.
- **Review mode keeps Telugu** (no toggle there in this version).
- **Toggle hidden (not broken) for two subjects that test the language itself:** English subject (344 questions — questions already ARE in English, explanation in Telugu) and Telugu subject (25 questions — questions stay in Telugu, explanation in Telugu).
- **Where the English comes from:** AP/TS questions use their ORIGINAL English stem + options from the source papers (recovered by joining to merged_unified.json, validated by option count + answer index + stem/option correspondence, including sequential matching for duplicate-Q papers such as TSPSC Group-IV 2018+2023). Sandeep questions had no English original, so their English versions were translated from Telugu (answer order, numbers, dates and names preserved). Two clean-ups were applied to AP/TS English only, and are logged in work/english/cleaning_log.json and work/english/header_cleaning_log.json: (a) 56 entries had source-header junk leaked from the source PDFs (e.g. "TSPSC … – Subject-wise Questions" inserted mid-stem/mid-option, and one "Summary Subject Questions …" index dump) — those header substrings were stripped; (b) 202 stems had source serial numbers absent from the Telugu stem (e.g. "373 I.", trailing ", 402", "[Paper Q.37]") — those serials were stripped. Question wording itself was not changed. A few English stems remain truncated in the source itself (e.g. s03-c42#19 "Jwala Mukhi is a" — the Telugu stem carries the same header junk, so the source question is incomplete in both languages); nothing was invented to complete them.

**English coverage (with English / total):** Polity 974/974, History 952/952, Telangana Specific 1,153/1,153, Geography 406/406, Environment 122/122, Economy 307/307, Current Affairs & GK 608/608, General Science 304/304, Biology 133/133, Physics 53/53, Chemistry 66/66, Reasoning & Quantitative 302/302, AP Specific 49/49, Education 27/27, Ethics/Society 39/39, English 0/344 (already English, toggle hidden), Telugu 0/25 (tests Telugu, toggle hidden). **Total with an English version: 5,495 / 5,864; without a separate English version: 369, all of them English/Telugu subjects by design — no other question is left without English.** (The toggle's disabled "English version ledu" state exists for robustness but should not occur for any non-English/Telugu question in this build.)

**Home footer:** the app home page has NO footer (the source/answer-count, excluded-questions and keyboard paragraphs were removed at the user's request; the detailed counts live only in this README).

**Per-question source lines (2026-11-04, user's request):** no source / answer-source line is shown under any question, and no source block is shown at the end of explanations (review mode included). The src/ansSrc fields remain in questions.js data but are not displayed in the app.

## Style (2026-11-04, restyle — data and flow unchanged)

Teal-navy-amber palette (header/hero teal-navy with a subtle diagonal pattern, solid teal primary buttons, amber/orange secondary, green success), dotted page background, rounder cards with softer shadows, and a subject icon on every subject card (e.g. Polity 🏛️, History 🏺, Geography 🌍). Layout, subject-first flow, syllabus chapters, questions, answers and explanations are unchanged.


## Design v3 (2026-11-04, design change — data and flow unchanged)

Warm paper background, maroon-gold-indigo palette, clean hero card with gold left border (no gradient banner), pastel subject tiles with large icons, pill set buttons, and letter-badge (A/B/C/D) option buttons in quizzes. Layout, subject-first flow, syllabus chapters, questions, answers and explanations are unchanged.

## Features (from the earlier quiz apps)

Practice mode (instant answer + explanation), Exam mode with 15/30/45-minute timer,
mark-for-review, optional −0.25 negative marking, question/option shuffle, Mixed 30,
Quick 10, Daily 30, Wrong-question revision, Bookmark revision, results ring + confetti
at ≥80%, review-all with explanations, share, XP / streak / badges, dark mode, font size,
sound, keyboard (1–4, N, P, M), resume of an interrupted quiz, phone-friendly fixed
(non-jumping) quiz menu bar and ☰ drawer with palette / Prev / Next / Submit / Home / Settings.
Progress is stored only in your own browser under new keys
`allSubjectsQuizV1` / `allSubjectsResumes` (paused quizzes list; the old single
`allSubjectsResume` key is migrated into it on load) — it does **not** touch
the geography (`geographyQuizV1`-style) or science (`scienceQuizV1`) app data.

## New features (2026-11-10)

**1. Back-button navigation:** the Android/iOS back button now follows
Home → Subject chapters → Quiz → Result backwards (Quiz → Subject page,
Subject → Home; Home back exits as usual). The quiz menu "⬅️ వెనక" button
also goes back to the subject page (the drawer "🏠 Home" button still goes
home directly). A quiz left with the back button is saved and can be resumed
from the home page's "⏸️ ఆగిన క్విజ్‌లు" list (every paused quiz listed with
its own Resume button).

**2. Spaced revision (📚 రివిజన్, new key `allSubjectsRevisionV1`):** every
question answered wrong (practice mode, exam mode at submit, or self-marked
"రాలేదు") enters the revision queue at Day 1, due the **next day**.
Answering it correctly moves it on: Day 1 correct → Day 4 (due in 4 days),
Day 4 correct → Day 7 (due in 7 days), Day 7 correct → Day 30 (due in 30
days), Day 30 correct → removed from the queue (Mastered). A wrong answer
at any stage keeps the question in the **same** stage, due tomorrow. The
home "📚 రివిజన్" card shows how many are due today; inside, each stage
lists its due questions grouped subject-wise with chapter names, and "Start
revision" runs a practice-style quiz of that stage's due questions only —
no submit, no XP, no re-entry into the wrong list; a correct answer
immediately advances the question (toast names the next stage) and a wrong
answer stays in the same stage.

**3. My Results (📊 నా ఫలితాలు, new key `allSubjectsResultsV1`):** every
submitted quiz (practice and exam) saves date, subject, chapter, set,
mode, score and % — last 100 kept, latest first, with a per-subject
summary (attempts, best %, average %) and a clear-history button. Results
live only on this device/browser and are lost if browser data is cleared.
Existing per-set best % (`allSubjectsQuizV1` progress) is untouched.

**4. "MR" logo:** the user's own image — silver serif MR in a thin circle on
black — in the header (circular) next to the title, as `favicon.png`, and as
home-screen icons `icon-192.png` / `icon-512.png` with `manifest.json` (name
"All Subjects Quiz", short name "MR Quiz", theme/background black, display
standalone).

## GitHub Pages steps (free)

1. Create a **public** GitHub repository.
2. Copy these **10 files to the repository root**: `index.html`, `questions.js`,
   `concepts.js`, `explanations.js`, `README.md`, `.nojekyll`, `favicon.png`,
   `icon-192.png`, `icon-512.png`, `manifest.json`.
   **Important:** `explanations.js` is ~32 MB — GitHub web upload fails at
   25 MB per file, so upload with **GitHub Desktop or `git push`** instead.
3. GitHub → repository **Settings → Pages** → Source: **Deploy from a branch** →
   Branch: **main**, Folder: **/ (root)** → Save.
4. Open the Pages URL shown there. Cost: **₹0** — no login, no backend, no paid service.

## Counts — per subject (total / printed key / official key (searched) / searched-verified / still open / chapters / sets)

| Subject | Total | Printed | Official (searched) | Searched/verified | Open | Chapters | Sets |
|---|---:|---:|---:|---:|---:|---:|---:|
| Polity | 974 | 689 | 56 | 224 | 5 | 23 | 44 |
| History | 952 | 539 | 57 | 338 | 18 | 27 | 44 |
| Telangana Movement | 502 | — | — | — | 13 | 24 | 30 |
| Telangana History | 311 | — | — | — | 6 | 11 | 16 |
| Telangana Culture | 256 | — | — | — | 4 | 14 | 17 |
| Telangana Geography | 225 | — | — | — | 7 | 1 | 8 |
| Telangana Economy & Schemes | 401 | — | — | — | 2 | 1 | 14 |
| Telangana Government & Administration | 61 | — | — | — | 0 | 2 | 3 |
| Geography | 406 | 97 | 57 | 245 | 7 | 12 | 19 |
| Environment | 122 | 51 | 10 | 60 | 1 | 6 | 7 |
| Economy | 307 | 75 | 26 | 196 | 10 | 10 | 14 |
| Current Affairs & GK | 608 | 128 | 97 | 381 | 2 | 10 | 26 |
| General Science | 304 | 93 | 42 | 164 | 5 | 5 | 12 |
| Biology | 133 | 19 | 29 | 84 | 1 | 8 | 9 |
| Physics | 53 | 7 | 12 | 33 | 1 | 7 | 7 |
| Chemistry | 66 | 8 | 10 | 46 | 2 | 7 | 7 |
| Reasoning & Quantitative | 302 | 131 | 77 | 90 | 4 | 15 | 18 |
| English | 344 | 92 | 74 | 176 | 2 | 10 | 17 |
| Telugu | 25 | 0 | 0 | 24 | 1 | 1 | 1 |
| AP Specific | 49 | 31 | 10 | 7 | 1 | 5 | 5 |
| Education | 27 | 27 | 0 | 0 | 0 | 1 | 1 |
| Ethics/Society | 39 | 0 | 3 | 35 | 1 | 5 | 5 |
| **Grand total** | **5864** | **2913** | **650** | **2215** | **86** | **205** | **305** |

> Telangana rows above show current totals (1,756 questions in all). The Printed / Official / Searched-verified splits were recorded only for the old combined subject (926 / 90 / 112) and have not been re-split per new subject. The Grand total row above is the earlier 5,864-question snapshot and is unchanged here.

Scored: **5,778** (answer 1–4 from the merged key, shown 0-indexed in the app).

## ⚠️ Telangana split (2026-11-06) — TG saved progress does NOT carry over

The single subject **Telangana Specific** (old chapter ids `s03-c01` … `s03-c53`) has been split into **6 subjects** with new chapter ids (`s18`–`s23`, see the mapping in `work/add3-pcb/tg_mapping.json`):

- **Telangana Movement** (`s18-c01`…`s18-c24`) — 24 chapters, 502 questions
- **Telangana History** (`s19-c01`…`s19-c11`) — 11 chapters, 311 questions
- **Telangana Culture** (`s20-c01`…`s20-c14`) — 14 chapters, 256 questions
- **Telangana Geography** (`s21-c01`) — 1 chapter, 225 questions
- **Telangana Economy & Schemes** (`s22-c01`) — 1 chapter, 401 questions
- **Telangana Government & Administration** (`s23-c01`…`s23-c02`) — 2 chapters, 61 questions

Old `s03-c53` “తెలంగాణ సమకాలీన అంశాలు” (Contemporary topics, 10 questions) is now **Telangana Government & Administration c02** (`s23-c02`).

**Saved progress: because TG chapter ids changed, saved set-progress, bookmarks and wrong-question lists for TG chapters do NOT carry over — the app starts fresh for TG only.** All other subjects' chapter ids are unchanged, so their saved progress is unaffected. No question, answer, option or question order changed — only the subject/chapter ids.
Open: **86** — shown with the badge "search chesina key dorakaledu — score lo ledu"
(or, where the note says so, "book key scan lo kanipinchaledu — score lo ledu"),
with the full reason under the question, excluded from scoring, no invented answer.

## How answers are labelled (never mislabelled)

Each question's explanation panel shows its source (paper/book + original question number,
exam tag / book page where available) and its answer source in Telugu:

- **printed key** — the key printed in the source paper/book itself.
- **official key (searched)** — an official key found by searching (not a printed book key).
- **searched/verified** — an answer found and verified by searching/solving (not a printed key).
- **still open** — no verifiable answer found; unscored.

A searched answer is **never** presented as a printed book key.

## What is NOT in this app (honest exclusions)

- **Excluded: 160 items** (kept out of the app entirely): 158 had no usable options
  (0 options in the source), 1 had option 4 polluted with a 2,200-character footer/key
  table so the real option 4 is unrecoverable (despite a known key), 1 had options not
  visible in the scan (despite a known key). One repair was possible and is included:
  TS Maths Q129 option 4 was truncated at an "Answer Key" marker and was repaired by
  truncating there.
- **v1-excluded, ~151 questions**: the scanned AP Constable Prelims 2018 Telugu paper
  questions, included in the subject PDFs only as scanned page images, never retyped —
  so they are not in this app.
- **Parser-missing: 10 questions** listed in Phase 1 (could not be parsed into the
  merged file) — also not in this app.
- **Reconciliation gap:** AP/TS detected 4,352 + Sandeep 1,682 + scanned ~151 = 6,185,
  vs delivered subject-PDF totals 6,183 — a gap of **2**, carried over from Phase 1
  counting; the app itself is verified at exactly 5,864 joined items (see below).

## English-subject exception

English questions and options **stay in English** — the English words themselves are the
test. Their explanations are in Telugu, like every other subject. All other subjects are
in Telugu, with English exam terms in brackets where the translation kept them.

## Phase-1 caveats (carried over, please book-check)

(a) The paper labelled "AP Constable Prelims, 6 Jan 2016 (Paper P2, Booklet A)" is in
fact **AP Constable 2018 Prelims PWT 06-01-2019 Paper P2**; its 155 answers are
searched/verified, but the board Final Key text could not be extracted — source labels
are kept as given, and these answers are flagged for the user's book check.
(b) The **S2 key is preliminary** — the final key after objections could differ on a few
questions.
(c) **Group-3 answers are single-attested** (17 anchors re-checked) — treat as
searched/verified, not as a printed key.
(d) **Mains 2025 answers use the Adda247 Set-B key**, spot-verified (English Q1–30
matched 30/30), with one PIB override: Q152 agricultural workers **263.1m per PIB**.

## Explanation depth — stated honestly

Grand mean across the 5,778 scored explanations is **101.8 words**, but per-slice depth
is uneven: slice01 Polity AP/TS **72.4**, slice09 Economy **80.2**, slice03 History AP/TS
**97.1**, all other slices **99.7–117.4**. So slices 01 and 09 are thinner than the rest;
a future expansion pass could even them out. Nothing was padded to hide this.

## Subject-lesson explanations (2026-11-04 — all 7,686 rewritten, supersedes the above)

The old explanations had a boilerplate problem: only 302 questions in 137 exact-duplicate
groups, but the same topic paragraph (e.g. one Constitution paragraph) was pasted into many
different Polity questions regardless of what each question asked, and Telangana Specific was
worst — 51 exact-duplicate groups (115 questions) plus generic study-advice sentences repeated
419/419/383/383/326 times. All 7,686 scored explanations were therefore rewritten fresh from
each question's own concept as **subject lessons**: correct answer + deciding reason → a concept
lesson (definition, origin/background, how it works, types, characteristics, examples,
distribution, related exam facts) → specific reasons for each wrong option (only where certain)
→ a question-specific exam/confusion point. Generic study advice ("read dates and names
together", "make a table") is banned as a substitute for a lesson; the banned TG sentences are
0 in the new set. No question, option, answer, chapter, or stem changed; open questions (105)
still have no explanation and are unscored.

- **Bold — final standard (important words only):** bold ONLY genuinely important words —
  correct answer name, key concept terms, critical numbers/dates/percentages, deciding
  place/person names, distinguishing contrasts. Never a whole sentence, never generic words
  alone. Distribution: mean **13.14** bold phrases per explanation (min 1, max 95), 0 phrases
  >8 words, 0 unmatched `**`, 72.5% of points contain an important bold phrase. A point with
  no important word is left unbolded — nothing is bolded merely to fill a point.
- **Length — do-not-reduce enforced:** new mean **176.7 words** (old 101.0), min 70, max 387;
  **0 explanations shorter than the old text they replaced**. 341 explanations are under 120
  words — all previously accepted dense pair/article lessons (Geography 268, Polity 72,
  TG Economy 1), not generic text. Per-subject new means: Polity 172.4, History 158.9,
  Telangana 201.1, Geography 148.9, Environment 193.9, Economy 178.2, Current Affairs 151.5,
  General Science 186.4, Biology 160.4, Physics 175.9, Chemistry 178.1, Reasoning 150.6,
  English 191.1, Telugu 213.2, AP Specific 205.0, Education 242.9, Ethics 258.5.
  (Reasoning note: the first Reasoning draft (mean 198.0) was rejected at merge audit —
  all 298 explanations were built on slot-frames that asserted answers instead of deriving
  them, including a profit-loss concept wrongly injected into interest questions. It was
  rewritten in five parts as real step-by-step derivations — series rules, formulas with
  substitution, Venn logic, placement chains, coding mappings — mean 150.6, 0 new < old,
  banned frames 0. The frame padding is gone, which is why the mean fell.)
- **Telangana Specific — research-expanded (priority subject):** 1,724 explanations, mean
  106.9 → **201.1** words, exact duplicates 0, all banned generic sentences 0. TG lessons were
  expanded from reliable sources (schemes: objective/launch/beneficiaries/amount/department;
  movement: full chronology; history: reign/capital/works; culture: place/origin/persons;
  TG geography/economy: district/river/soil specifics). Per TG slice: Movement & Government
  (tg_a) 146.6, TG History & Culture (tg_b) 232.8, TG Geography (tg_c) 210.4, TG Economy &
  Schemes (tg_d) 207.2, TG Government & programmes (tg_e, deepened) 275.0. Facts that could
  not be verified were left general, not invented, and doubtful keys keep an explicit
  in-text flag (e.g. TG Government Q21: keyed MeeSeva 2.0 vs the public Bhu Bharati
  Dharani-replacement fact; TG Panchayat Raj Q1: keyed quorum 300 vs the Act's 200 for
  3,001–5,000 voters) — keys were never flipped by an explanation.
- **Boilerplate audit (new set):** 0 exact-duplicate explanations; no sentence (≥8 words)
  appears in >5 explanations except answer-naming sentences (correct combination/order and
  assertion–reason verdicts); masked-template audit shows only previously disclosed scaffolds
  (a Reasoning closing recap, Polity "which is NOT correct" distractor logic, TG membership
  logic) — no lesson-content template.
- **Honest limitations:** some slices miss the 160–200 mean on density grounds (History 158.9,
  Geography 148.9, Current Affairs 151.5; TG Movement & Government slice 146.6) — padding them
  would violate the no-invention rule. A minority of TG History/Economy explanations use a
  criterion-only distractor sentence where a wrong option's distinguishing fact could not be
  verified (none invented), and a minority of TG Economy explanations have lesson + exam point
  without a distractor section where no verifiable specific reason existed. Reasoning
  diagram/table-dependent questions teach the method from the stem where possible and are
  flagged. All flagged keys from the rewrite (dozens, e.g. conflicting density orders, the
  Bhu Bharati/MeeSeva conflict, first-patent chromosome conflicts) still await a book check;
  no key was changed.

## Verification (build-time, before delivery)

- Join by qid: **5,864 / 5,864**, 0 duplicates, 0 missing; every translated item used once.
- Scored MCQ validity: all 5,778 have 4 options, source answer 1–4, app answer 0–3,
  answer_source present; all 86 open items have answer null, a non-empty open_note and
  no explanation (so no invented answer can leak in).
- Bold checks on all 5,778 scored explanations: **2–8 bold phrases each, ≤8 words each,
  0 generic-only bold, 0 unmatched `**` markers**; stripping `**` reproduces the text.
  Open explanations carry **0 bold**.
- jsdom DOM test on the single-file preview: subject grid renders all 17 subjects +
  All-subjects card; choosing a subject filters topics to that subject only; a sample MCQ
  explanation renders as points-per-line with bold and no `**` leak; an open question
  shows the correct badge/note and is unscored; sets of 30 are correct.

- Topic-wise regroup verification (2026-11-04): all 5,864 questions remapped exactly once
  (old paper-wise topic id + position → new chapter); identity multiset (stem, options, answer,
  type, source, answer_source) old vs new is identical; explanations remapped to new chapter keys,
  still 5,778, 0 for open; NO paper/exam name appears in any of the 205 chapter titles
  (grep for Constable / Group- / SI Prelims / AEE / Booklet / Set A/B: 0 hits — paper names
  survive only inside question source lines); General Science "ఇతర విషయాలు" had 0 questions and
  was dropped, and Reasoning gained two chapters (జ్యామితి & కొలతలు 18, బీజగణితం & సంఖ్యా వ్యవస్థ 18)
  to bring its fallback from 39 (12.9%) down to 3 (1.0%) — total chapters 205, sets 305.
- jsdom DOM test on the new preview: initial 18 subject cards / 0 chapter cards; choosing Polity
  shows only Polity chapters, and the ప్రాథమిక హక్కులు chapter contains questions from 19 distinct
  source papers; choosing History shows only History chapters; All-Mixed shows 0 chapter cards;
  clearing returns to 0 chapter cards; a quiz from a syllabus chapter shows its per-question
  explanation with bold and no `**` leak.

## Files in this folder

`index.html` (app + subject-first, chapter-wise home), `questions.js` (205 chapters, 7,940 questions),
`explanations.js` (7,835 scored explanations, `**` bold markers), `concepts.js`
(empty CONCEPTS/CONCEPT_MAP fallback — per-question explanations exist, so no separate
concept entries are needed), `README.md` (this file), `.nojekyll` (GitHub Pages),
`favicon.png` (MR logo), `icon-192.png` / `icon-512.png` (MR home-screen icons),
`manifest.json` (PWA manifest: name "All Subjects Quiz", short name "MR Quiz").

## Syllabus chapters — full list (chapter : questions)

### Polity — 23 chapters, 974 questions, 44 sets

- భారత రాజ్యాంగం: 97
- ప్రాథమిక హక్కులు: 64
- ఆదేశిక సూత్రాలు: 30
- ప్రాథమిక విధులు: 27
- పౌరసత్వం: 8
- పార్లమెంటు: 40
- లోక్‌సభ: 28
- రాజ్యసభ: 24
- రాష్ట్రపతి - ఉపరాష్ట్రపతి: 47
- కేంద్ర ప్రభుత్వం: 38
- గవర్నర్: 26
- రాష్ట్ర ప్రభుత్వము: 40
- ఆర్టికల్స్: 43
- రాజ్యాంగ సవరణలు: 49
- రాజ్యాంగంలోని షెడ్యూల్స్ & భాగాలు: 36
- పంచాయతీ రాజ్ వ్యవస్థ: 61
- పట్టణ స్థానిక ప్రభుత్వం: 21
- భారత ఎన్నికల సంఘం: 44
- చట్టాలు-అమలు: 36
- కమిటీలు & కమిషన్లు: 59
- న్యాయ వ్యవస్థ: 75
- కేసులు: 17
- ఇతర విషయాలు: 64

### History — 27 chapters, 952 questions, 44 sets

- బౌద్ధ, జైన, సిక్కు మతాలు: 34
- సింధూ నాగరికత: 14
- ప్రాచీన చరిత్ర & వివిధ అంశాలు: 84
- మౌర్యులు: 18
- గుప్తులు: 17
- పల్లవులు - చాళుక్యులు: 14
- ప్రాచీన భారతదేశ రచయితలు: 34
- ఢిల్లీ సుల్తానులు: 29
- మొగల్స్: 29
- యుద్ధాలు: 23
- విజయనగర & బహమనీ సామ్రాజ్యాలు: 29
- శివాజీ, పీష్వాలు: 11
- భక్తి, సూఫీ ఉద్యమాలు: 16
- భారతదేశ ఆక్రమణ : యూరోపియన్ల రాక: 14
- సాంఘిక సంస్కరణోద్యమాలు: 85
- రైతు, గిరిజన ఉద్యమాలు: 10
- బ్రిటీష్: 91
- 1857 తిరుగుబాటు: 25
- ఆధునిక భారతచరిత్ర - రచయితలు: 29
- పత్రికలు: 17
- సంస్థలు, సభలు, పార్టీలు: 47
- జాతీయోద్యమ నాయకులు: 49
- కాంగ్రెస్ పార్టీ: 32
- గాంధీజీ: 39
- భారత స్వాతంత్ర్య ఉద్యమం: 100
- ప్రపంచ చరిత్ర & అంతర్జాతీయ సంబంధాలు: 23
- ఇతర విషయాలు: 39

### Telangana Movement — 24 chapters, 502 questions, 30 sets

- తెలంగాణ సాంస్కృతిక పునరుజ్జీవనం: 25
- పత్రికలు – స్థాపకులు: 13
- పుస్తకాలు – రచయితలు: 20
- తెలంగాణ ఉద్యమం — (Topic name unreadable - source PDF page 6 corrupted): 10
- నిజాం రాజ్యం – భారతదేశంలో విలీనం: 16
- 1952-ముల్కీ ఉద్యమం: 9
- పెద్ద మనుషుల ఒప్పందం (1956): 16
- బూర్గుల ప్రభుత్వం: 10
- 1969-తెలంగాణ ఉద్యమం: 40
- నక్సలిజం: 10
- కమిటీలు/ కమిషన్లు: 45
- తెలంగాణ సాయుధ పోరాటం: 13
- ఉస్మానియా యూనివర్సిటీ: 8
- G.O. NO. 36 & 610: 9
- NTR & CBN పాలన: 6
- 5, 6 మరియు 8 సూత్రాలు పథకం: 11
- తెలంగాణ ఉద్యమ పాటలు: 13
- తెలంగాణ ఉద్యమ పార్టీలు & సంస్థలు: 40
- ప్రముఖ వ్యక్తులు: 30
- రాజకీయ పార్టీలు: 14
- తెలంగాణ మలిదశ ఉద్యమం: 67
- తెలంగాణ ఆవిర్భావం: 25
- ఆంధ్రప్రదేశ్ పునర్వ్యవస్థీకరణ: 11
- ఇతర విషయాలు: 41

### Telangana History — 11 chapters, 311 questions, 16 sets

- తెలంగాణ చరిత్ర — చరిత్ర పూర్వయుగం & మతాలు: 30
- తెలంగాణ చరిత్ర — ఇక్ష్వాకులు: 4
- తెలంగాణ చరిత్ర — విష్ణుకుండినులు: 9
- తెలంగాణ చరిత్ర — చాళుక్యులు: 12
- తెలంగాణ చరిత్ర — రాష్ట్రకూటులు: 3
- తెలంగాణ చరిత్ర — పద్మ, నాయకులు & వెలమలు: 10
- తెలంగాణ చరిత్ర — శాతవాహనులు: 30
- తెలంగాణ చరిత్ర — కాకతీయులు: 63
- తెలంగాణ చరిత్ర — కుతుబ్ షాహీలు: 50
- తెలంగాణ చరిత్ర — ఆసఫ్ జాహీలు: 57
- తెలంగాణ చరిత్ర — కవులు – రచనలు: 43

### Telangana Culture — 14 chapters, 256 questions, 17 sets

- తెలంగాణ సంస్కృతి — సృష్టికళలు / సంప్రదాయం: 27
- తెలంగాణ సంస్కృతి — తెలంగాణ కళలు: 36
- తెలంగాణ సంస్కృతి — పండుగలు: 33
- తెలంగాణ సంస్కృతి — నిర్మాణాలు: 37
- తెలంగాణ సంస్కృతి — దేవాలయాలు: 29
- తెలంగాణ సంస్కృతి — జాతరలు: 12
- తెలంగాణ సంస్కృతి — కవులు - రచనలు: 21
- తెలంగాణ సంస్కృతి — తెలంగాణ ప్రముఖులు: 10
- తెలంగాణ సంస్కృతి — తెలంగాణ కులాలు: 9
- తెలంగాణ సంస్కృతి — తెలంగాణ తెగలు: 13
- తెలంగాణ సంస్కృతి — సినిమాలు: 8
- తెలంగాణ సంస్కృతి — తెలంగాణ యాస/మండలికం: 5
- తెలంగాణ సంస్కృతి — జిల్లాలు పూర్వపు నామాలు & ప్రత్యేకతలు: 7
- తెలంగాణ సంస్కృతి — తెలంగాణ రాష్ట్ర అధికారిక చిహ్నాలు: 9

### Telangana Geography — 1 chapters, 225 questions, 8 sets

- తెలంగాణ భూగోళం: 225

### Telangana Economy & Schemes — 1 chapters, 401 questions, 14 sets

- తెలంగాణ ఆర్థిక వ్యవస్థ & పథకాలు: 401

### Telangana Government & Administration — 2 chapters, 61 questions, 3 sets

Note: old `s03-c53` “తెలంగాణ సమకాలీన అంశాలు” (Contemporary topics, 10 questions) is merged here as c02 (`s23-c02`).

- తెలంగాణ ప్రభుత్వం & పరిపాలన: 51
- తెలంగాణ సమకాలీన అంశాలు: 10
### Geography — 12 chapters, 406 questions, 19 sets

- స్థానం & విస్తరణ: 18
- భౌతిక స్వరూపం: 74
- వాతావరణం: 41
- నదులు & నీటి వనరులు: 44
- నేలలు: 13
- వ్యవసాయం: 27
- ఖనిజాలు & పరిశ్రమలు: 35
- జనాభా & జనగణన: 30
- రవాణా & వాణిజ్యం: 21
- ప్రపంచ భూగోళం: 38
- ఆంధ్రప్రదేశ్/తెలంగాణ భూగోళం: 59
- ఇతర విషయాలు: 6

### Environment — 6 chapters, 122 questions, 7 sets

- పర్యావరణ వ్యవస్థలు: 9
- జీవవైవిధ్యం: 13
- కాలుష్యం: 26
- వాతావరణ మార్పు: 15
- విపత్తు నిర్వహణ: 38
- పర్యావరణ చట్టాలు & సంస్థలు: 21

### Economy — 10 chapters, 307 questions, 14 sets

- పరిచయం & GDP: 49
- బ్యాంకింగ్ & ద్రవ్య విధానం: 38
- బడ్జెట్ & పన్నులు: 23
- ప్రణాళిక & నీతి ఆయోగ్: 17
- వ్యవసాయం & పరిశ్రమలు: 46
- దారిద్ర్యం & నిరుద్యోగం: 18
- ప్రభుత్వ పథకాలు: 43
- అంతర్జాతీయ వాణిజ్యం: 25
- TG/AP ఆర్థిక వ్యవస్థ: 29
- ఇతర విషయాలు: 19

### Current Affairs & GK — 10 chapters, 608 questions, 26 sets

- జాతీయ వార్తలు & నియామకాలు: 187
- అంతర్జాతీయ వార్తలు: 120
- క్రీడలు: 62
- పురస్కారాలు: 86
- పుస్తకాలు & రచయితలు: 15
- శాస్త్ర సాంకేతికత: 41
- రక్షణ: 31
- దినోత్సవాలు: 16
- సంస్థలు & ప్రధాన కార్యాలయాలు: 32
- ఇతర GK: 18

### General Science — 5 chapters, 304 questions, 12 sets

- భౌతిక శాస్త్రం: 58
- రసాయన శాస్త్రం: 42
- జీవశాస్త్రం: 97
- శాస్త్ర సాంకేతికత & ఆవిష్కరణలు: 79
- పర్యావరణ శాస్త్రం: 28

### Biology — 8 chapters, 133 questions, 9 sets

- మొక్కలు: 36
- జంతువులు: 8
- మానవ శరీరం: 23
- వ్యాధులు: 15
- విటమిన్లు & పోషకాలు: 10
- జన్యుశాస్త్రం: 18
- పర్యావరణ జీవశాస్త్రం: 9
- ఆరోగ్యం: 14

### Physics — 7 chapters, 53 questions, 7 sets

- కాంతి: 10
- ఉష్ణం: 6
- ధ్వని: 6
- విద్యుత్ & అయస్కాంతత్వం: 11
- యాంత్రిక శాస్త్రం: 16
- ఆధునిక భౌతిక శాస్త్రం: 1
- కొలతలు & పరికరాలు: 3

### Chemistry — 7 chapters, 66 questions, 7 sets

- పరమాణు నిర్మాణం & మూలకాలు: 18
- ఆమ్లాలు/క్షారాలు/లవణాలు: 17
- లోహాలు: 11
- కర్బన రసాయన శాస్త్రం: 7
- రసాయన ప్రతిచర్యలు: 4
- రోజువారీ జీవితంలో రసాయన శాస్త్రం: 3
- ఇతర విషయాలు: 6

### Reasoning & Quantitative — 15 chapters, 302 questions, 18 sets

- సంఖ్యా శ్రేణులు: 37
- శాతాలు & నిష్పత్తులు: 17
- సగటు: 9
- లాభనష్టం & వడ్డీ: 17
- కాలం & పని: 8
- వేగం & దూరం: 6
- కోడింగ్-డీకోడింగ్: 25
- దిశలు: 14
- రక్త సంబంధాలు: 10
- తర్కం & ప్రకటనలు: 76
- డేటా ఇంటర్‌ప్రిటేషన్: 29
- గడియారాలు & క్యాలెండర్: 15
- జ్యామితి & కొలతలు: 18
- బీజగణితం & సంఖ్యా వ్యవస్థ: 18
- ఇతర విషయాలు: 3

### English — 10 chapters, 344 questions, 17 sets

- పదజాలం — Synonyms/Antonyms: 69
- స్పెల్లింగ్ (Spelling): 16
- వ్యాకరణం — Tenses/Articles/Prepositions: 67
- Fill in the Blanks: 62
- One Word Substitution: 11
- Idioms & Phrases: 30
- Comprehension: 42
- Sentence Correction/Arrangement: 24
- Active/Passive & Direct/Indirect: 21
- ఇతర విషయాలు: 2

### Telugu — 1 chapters, 25 questions, 1 sets

- సాధారణ తెలుగు: 25

### AP Specific — 5 chapters, 49 questions, 5 sets

- AP చరిత్ర & సంస్కృతి: 13
- AP భూగోళం: 23
- AP ఆర్థిక వ్యవస్థ & పథకాలు: 2
- AP పునర్వ్యవస్థీకరణ: 7
- AP సమకాలీన అంశాలు: 4

### Education — 1 chapters, 27 questions, 1 sets

- విద్య (Education): 27

### Ethics/Society — 5 chapters, 39 questions, 5 sets

- నైతికత & విలువలు: 3
- సమాజం & లింగం: 12
- సంక్షేమం & విద్య: 6
- భావోద్యోగిక మేధస్సు: 13
- జనాభా & తెగలు: 5


## NoFiller rewrite 2026-11-09 — Polity, Telangana History, Telangana Movement explanations (1,803 keys)

User's instruction: Polity / Telangana History / Telangana Movement explanations had too much filler — every line must teach NEW matter, nothing should be filler, and the explanations alone must complete the syllabus. All 1,803 scored explanations in these three subjects were rewritten fresh from each question's concept (old text NOT copied): Polity (s01) 999, Telangana History (s19) 307, Telangana Movement (s18) 497. Open questions (24 in these subjects, 105 in the app) keep explanation null. Questions, options, answers, chapters and ids unchanged; only explanations.js and this README changed (questions.js and index.html byte-identical to the ExamMatch build).

- Word counts (new ≥ old for every key, 0 below): Polity mean 204.0 → 227.7 (min 150, max 366), TG History 245.9 → 274.6 (min 200, max 385), TG Movement 166.6 → 193.7 (min 133, max 278). Overall mean 222.3.
- Bold (important words only, each phrase ≤8 words): Polity mean 32.6 (min 11, max 64), TG History 44.3 (min 21, max 74), TG Movement 22.5 (min 11, max 40). 0 phrases >8 words, 0 unmatched `**`, 0 exact duplicate explanations.
- Filler audit (verified by script + full reading of 110 explanations): 0 flagged lines (no line whose bold facts were all already stated earlier in the same explanation), 0 pure-recap memory lines, 0 sentences (≥8 words) repeated in >5 explanations. Old filler removed, e.g. Polity s01-c01#1's answer restated three times and empty distractor shells; TG Movement s18-c21#1's same answer sentence four ways plus a generic "remember this" line.
- Answer-support sample (120 keys, 40/subject): 114/120 by strict token match; the 6 "failures" are token-matching artifacts (option-code answers, transliterated option text) — all 6 manually verified as actually supported, so true support is 120/120. Distractor-addressed 105/120.
- Direct-match check (10 PCB-paper questions in these subjects): 9 direct + 1 direct-by-elimination = 10/10 answerable. Q75 (municipality types vs 243Q(1)(a)/(b)/(c)) and Q105 (Telangana formation, statement 3) were partial at first check and were fixed by adding the missing facts.
- Factual corrections made in explanations only (keys NEVER flipped): AP Legislative Council strength 50 → 58 (2015 Amendment Act, w.e.f. 29-4-2015) in 10 keys; AP mandals 1104 → 680 (2022 reorganisation); Prakasam = first CM of Andhra State (1953), Neelam Sanjiva Reddy first AP CM (1956); K.K. Venugopal appointed AG at 86, not 91; Dhankhar resigned 21 July 2025, C.P. Radhakrishnan RS Chairman from 12 Sept 2025; Rudramadevi = first woman ruler of the Kakatiya dynasty (not of the subcontinent); Mir Jumla left Golconda 1656 — Madanna/Akkanna were appointed by Tana Shah; Shoebullah Khan was the Imroze editor murdered by the Razakars in Aug 1948 (not a Razakar leader); HCS founded 1882 by Salar Jung I; Tripura literacy figures corrected (2011: Tripura 87.22%, Mizoram 91.33% second among states, Goa 88.70%) — the printed key (Tripura) is kept, with a book-check flag, because official figures rank Goa above Tripura among the options.
- Honest limits: TG Movement mean 193.7 is below the 200–230 target (dragged by the c17–c24 slice, mean 178.4); TG History mean 274.6 is above target because new≥old forced it up. Cross-explanation repetition of canonical facts (e.g. RS strength 250/238/12, Art 12 definition) is deliberately kept — zero-filler applies within one explanation, not across the bank. PCB Q103's Second Vishalandhra Mahasabha date is not in the app and could not be verified — not invented. Flagged keys (never flipped) await the user's book check: PCB Q57/Q60/Q70, s16-c01#13, s01-c20#17 (Law Commission 2012–2015 vs 2013–2015), s01-c23#48 (Mahakumbh commission), s01-c13#8 (key order), s18-c19#13 (Regional Committee 1969 vs 1958), s18-c18#5 vs #36 (Telangana Convention founder), Ramappa tank (Ganapatideva vs Recharla Rudra), s01-c23#58 (Tripura literacy key vs official figures).
- Upload note: explanations.js is now larger than 30.6 MB — upload via GitHub Desktop or git push (GitHub web upload limit is 25 MB per file). Totals unchanged: 7,940 questions (7,835 scored + 105 open), 205 chapters, 22 subjects, 368 sets.
"# MR-quiz-app" 
