# Write-up

MATACS-TH-2026-02 · Step numbers refer to the [process diagram](diagram/process-flow.png).

**The idea in one line:** the AI only *reads*, pulling out facts with the exact sentence as proof. Plain code does every pass/fail, score and ranking. A person checks the result and makes the award.

## 1. AI vs. rules

- **AI, one step:** step 3 reads each proposal and returns, for every fact the admin's criteria need, a value plus the exact sentence and page it came from, or "not stated".
- **Plain code, everything else:** reading the file and scanning for lines aimed at an AI (2), checking the AI's answer (4), the suspicious/wrong-category decision (5), must-haves (6), points (7) and ranking (8).
- **Why the line is there:** proposals say the same thing in different ways. B mentions a "licensed subcontractor", so a keyword search would wrongly pass it; working out *who* holds the licence needs a language model. Once the facts are known, pass/fail, maths and sorting are exact in code, and a model would only add variation. It also means a proposal that fools the AI still can't give itself points.
- **Trade-off I considered:** two AI passes (must-haves first, scoring facts only for bids that pass). I chose one: reading the document is most of the cost, most bids pass, and a reviewer who overturns a rejection needs the full facts anyway. With hundreds of bids or very long documents, two passes would be worth it.

## 2. Consistency and explainability

- **Same input, same ranking:** each AI answer is saved, tied to the file's fingerprint, the questions, the prompt version and the model. Re-ranking reuses the saved answers, so it gives an identical result. Current models don't offer "temperature 0", so a *fresh* extraction could differ on a borderline fact. Step 4 and the reviewer catch that, and the log shows which answer was used.
- **Fixed formulas (step 7),** the same for every bid:
  - Higher is better (experience, warranty years): points × value ÷ best value among eligible bids.
  - Lower is better (price, weeks): points × best value ÷ value. Over the admin's limit ($260K, 16 weeks) scores 0.
  - Checklist (team and approach): an equal share per item found: named lead, crew, safety plan, monitoring plan.
  - Not stated scores 0, with a "missing" note.
- **Explainable:** for each proposal the reviewer sees points per criterion, each next to the sentence and page it came from. The one-line reason is built from those numbers, not written by the AI.

## 3. Human in the loop

- **Step 9, reviewer:** sees the ranked list plus every flagged and ineligible proposal with its reason. They can correct a fact, clear a flag or override a result. Each change needs a written reason and sends the proposal back through the must-have check (step 6).
- **Step 10:** a person makes the award. The tool only recommends.
- **Admin:** criteria changes are a human action too, and are versioned.
- **Audit log, append-only:** file fingerprint, uploader and time; criteria version; model and prompt version; the AI's raw answer with quotes; what step 4 rejected and why; points per criterion; status and rank; every human change with who, when and why. Any score can be traced back to a sentence in a file.

## 4. Changing criteria

- Criteria are never edited in place. Saving creates a new version, and each run is locked to the version it started with (step 2). Old results stay explainable and are marked "out of date".
- **Weights or limits changed:** re-score from the saved AI answers. No new AI call, so it is instant and free.
- **Criterion or question added or changed:** the AI reads the proposals again, because the saved answers don't cover the new question.
- Both versions stay in the log, so nobody can quietly change the rules to favour one bid after seeing the results.

## 5. Risks and safeguards

| Risk | Safeguard |
|---|---|
| Wrong extraction (misread number) | Every value needs a quote. Step 4 checks the quote is in the file and the number appears in it. The reviewer sees quote and score side by side |
| Made-up facts | "Not stated" is always allowed and scores 0. A quote not found in the file throws the value out |
| Manipulation (planted instructions, like C) | Proposal passed to the AI as data inside tags; a code phrase scan plus the AI's own report flag it; the AI can't award points anyway |
| Bias (brand, writing style, company size) | AI extracts facts, not opinions, and never sees other bids or scores; the same formulas apply to everyone |
| Unreadable or wrong files | OCR, otherwise flagged. Wrong category is flagged |
| Relative scoring (one bid shifts others' points) | Only eligible, unflagged bids set the "best" value; the reviewer sees the formula and breakdown |
| Cost | One AI call per proposal, saved and reused; re-scoring is free; duplicate files skipped by fingerprint |
| Over-trust (rubber-stamping) | Labelled a recommendation; flagged and ineligible bids are always shown; overrides need a reason |

## Assumptions behind [ranking.json](ranking.json)

- **B:** a licence held by a subcontractor "to be confirmed" does not meet "held by the bidder".
- **D:** "schedule to be determined" counts as not stated (0 points), not ineligible, because the admin made timeline a weighted criterion, not a must-have.
- **C:** "since 2020" may include installs older than 5 years, so the count is kept with a note for the reviewer. Weekly progress reports don't count as a performance-monitoring plan.
- Warranty points are split equally between panel and workmanship years. A price over budget or a timeline over 16 weeks scores 0 on that criterion with a note, not automatic rejection.
