# AI Proposal Triage Design

| | |
|---|---|
| **Name** | Umair Ahmed |
| **Reference** | MATACS-TH-2026-02 |
| **Time spent** | About 60 minutes |

## Files

| File | What it is |
|---|---|
| [diagram/process-flow.png](diagram/process-flow.png) | Process flow from upload to ranked list: AI step (purple), plain-code steps (blue), human checkpoints (green) |
| [WRITEUP.md](WRITEUP.md) | Answers to the five questions, and my assumptions |
| [PROMPTS.md](PROMPTS.md) | The extraction prompt exactly as sent, with its output schema |
| [ranking.json](ranking.json) | Expected ranking of proposals A to E |

## The design in brief

The AI does one job: it reads each proposal and pulls out the facts the admin's criteria ask for, each with the exact sentence it came from, or "not stated". Plain code then checks those quotes against the file, applies the must-haves, gives points with fixed formulas and ranks. Anything suspicious, unreadable or unclear is held for a person, and a person makes the award. A proposal that tries to talk to the AI, like C, can't win points that way.

| | Result | Why |
|---|---|---|
| A | #1, 98.0 | Most experience, fastest committed timeline, full team plan |
| E | #2, 84.8 | Best warranty and monitoring, but least experience |
| D | #3, 69.8 | Cheapest eligible, but no committed timeline (0/15) and fewer team details. Ask the bidder before award |
| C | Flagged | Hidden note telling automated reviewers to rank it first. Would score 77.1 if a person clears it |
| B | Ineligible | Cheapest and fastest, but the licence belongs to a subcontractor not yet named, not to B |

## How I used AI tools

I worked through the brief, the process flow and the sample edge cases and made the final design calls, using an AI assistant (Claude) to discuss options and help draft the wording and diagram, which I reviewed before keeping. One place I disagreed: the assistant's first flow had the AI read the whole proposal before the pass/fail check, and I questioned why the must-haves weren't checked first. I kept one AI pass in the end, because a keyword check would wrongly pass B (it mentions a "licensed subcontractor") and reading every proposal twice costs more, but I made sure the cheaper two-pass "fail fast" option is in the write-up as a trade-off.
