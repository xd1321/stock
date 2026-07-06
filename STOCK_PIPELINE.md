# Stock Research Pipeline

A 3-stage funnel: Red Flag Sweep -> Quality Gate -> Catalyst Check.
Each stage only runs if the previous one passes. Kills weak picks fast,
spends deep research time only on survivors.

**Not financial advice.** This produces a structured research summary
based on your own criteria - it does not tell you what to buy.

---

## HOW TO RUN THIS

### One ticker, manually (in VS Code terminal or panel)
```
claude "Run the full stock research pipeline in STOCK_PIPELINE.md on ticker NOK. Only show me the final verdict and, if it survives all 3 gates, the 3 catalyst cards."
```

### A whole watchlist, manually
```
claude "Run the stock research pipeline in STOCK_PIPELINE.md on every ticker in watchlist.txt. Only output tickers that survive all 3 gates, as a ranked list with the REAL catalysts. Skip and don't report anything on tickers that fail."
```

### Automatically, every day (no manual trigger)
See "AUTOMATING THIS" at the bottom - Claude Code supports scheduled
tasks / routines that will run this on a cadence and hand you a
finished list without you opening a session.

---

## PROMPT 1 - RED FLAG SWEEP

```
Act as a skeptical short-seller researching {TICKER}.
I need a 90-second red-flag sweep - I'm trying to decide if this pick
is even worth more research.

Search SEC filings (or, for foreign private issuers, 6-K/20-F and
home-market equivalents), earnings transcripts, and recent news.

Check 5 things, each tagged HIGH / MEDIUM / LOW severity:
1. REVENUE TREND - YoY growth direction over last 4 quarters.
   Decelerating, flat, or declining? How fast?
2. MARGIN PRESSURE - Gross AND operating margin trend over last 4
   quarters. Compressing? Stable? Expanding?
3. COMPETITIVE THREAT - Named competitor taking material share. Name
   the competitor and the magnitude. "Industry dynamics" doesn't count.
4. INSIDER ACTIVITY - Unscheduled sells in last 90 days (Form 4 in the
   US; EU Market Abuse Regulation Article 19 manager transactions for
   foreign private issuers like Nokia). NOT 10b5-1 / pre-planned
   transactions - those aren't a signal. What's the net $ amount?
5. GUIDANCE PATTERN - Beats, misses, or cuts in last 4 quarters? Any
   guidance reductions? Hedging language on the most recent call?

Return the 3 most severe risks ranked HIGH > MED > LOW. Cite sources
for each - 10-K/6-K page, transcript quote, or filing URL.

DECISION RULE:
- 2+ HIGH severity -> SKIP. The pick is structurally broken.
- 1 HIGH + multiple MEDIUM -> SKIP unless you have a real edge on the
  bear case (most people don't).
- All LOW/MEDIUM -> move to Prompt 2.
```

## PROMPT 2 - QUALITY GATE (only if Prompt 1 passes)

```
Run a V1 Quality Gate on {TICKER}.
Search the web for current TTM data. Never estimate from memory.

Check 4 binary quality conditions:
- ROIC >= 15% (TTM): PASS / FAIL
- Positive Free Cash Flow (last 4 quarters): PASS / FAIL
- Net Debt / EBITDA < 2x: PASS / FAIL
- Revenue growing YoY (latest quarter): PASS / FAIL

Then check the QUALITY FLOOR - auto-fail conditions that override
partial passes above:
- Revenue declining 3+ consecutive quarters -> AUTO-FAIL
- Unscheduled insider sells > $50M last 90 days -> AUTO-FAIL
- Guidance cut in last 12 months -> AUTO-FAIL

OUTPUT:
- 4 binary checks with the actual numbers
- Quality Floor: PASS or AUTO-FAIL (and which trigger if AUTO-FAIL)
- Final gate verdict: PASS (3+ of 4 pass + floor passes) or FAIL

DECISION RULE:
- AUTO-FAIL on Quality Floor -> SKIP. Doesn't matter what the rest
  looks like. The business is decaying.
- 2+ Quality checks FAIL -> SKIP. Not enough business there to bet on.
- 3+ Quality checks PASS + Floor passes -> move to Prompt 3.
```

## PROMPT 3 - CATALYST CHECK (only if Prompt 2 passes)

```
For {TICKER}, identify the 3 primary bull-case catalysts in the next
12 months. The story that's supposed to make this stock go up.

For each catalyst, score:
- PROBABILITY - LOW / MEDIUM / HIGH (is this likely?)
- MAGNITUDE - LOW / MEDIUM / HIGH (does it actually move the stock if
  it hits?)
- TIMING - 0-6mo / 6-12mo / 12-18mo / Uncertain
- ALREADY PRICED IN - Y / N (does consensus already assume this is
  happening?)

Tag each catalyst as ONE of:
- REAL - high probability + meaningful magnitude + NOT already priced
  in + clear timing window
- HYPE - narrative without measurable proof points or evidence of
  inflection
- PRICED IN - consensus already assumes this; even if it hits exactly
  as expected, no re-rate

OUTPUT: Three catalyst cards, one per catalyst. Include the specific
event, evidence for the probability score, and the source.

DECISION RULE:
- 0 catalysts tagged REAL within 12 months -> SKIP. No reason to own
  this - it has nowhere to go.
- 1+ catalysts tagged REAL -> pick survived the filter. Now check the
  chart for entry.
```

---

## FINAL OUTPUT FORMAT (for batch runs across a watchlist)

```
SURVIVORS (passed all 3 gates)
-------------------------------
TICKER - [# of REAL catalysts]
  Top REAL catalyst: [event] | Timing: [window] | Source: [cite]
  Quality gate: [3/4 or 4/4 pass]
  Red flags noted: [worst one, severity]

SKIPPED
-------------------------------
TICKER - Failed at [Prompt 1 / 2 / 3] - reason: [1-line reason]
```

---

## AUTOMATING THIS (daily, hands-free)

Claude Code supports running a saved prompt automatically on a
schedule, so you don't have to trigger it yourself each morning.
Two options (check the current docs at
https://code.claude.com/docs/en/desktop-scheduled-tasks since these
features are still evolving):

1. **Desktop scheduled tasks (local)** - runs on your machine, only
   while Claude Code Desktop is open and your computer's awake.
   In any Desktop session: "Set up a daily task that runs the pipeline
   in STOCK_PIPELINE.md against watchlist.txt every morning at 8am,
   and only report tickers that survive all 3 gates."

2. **Routines (cloud)** - runs on Anthropic's infrastructure even if
   your laptop is off, on a cron-style schedule. Set up with
   `/schedule` in the CLI, e.g.:
   "Create a daily routine at 7am that runs the pipeline in
   STOCK_PIPELINE.md over watchlist.txt and writes survivors to
   results/YYYY-MM-DD.md."

Either way, add explicit guardrails to the prompt since a scheduled
run can't pause to ask you questions - e.g. "if data for a ticker
can't be found, skip it and note that in the output" rather than
leaving it to guess.

---

## STATUS: ACTIVE CLOUD ROUTINE

A cloud routine ("Daily Stock Watchlist Pipeline") runs this funnel
every day at 8am America/Toronto (12:00 UTC / cron `0 12 * * *`)
against NOK. It cannot see this file or watchlist.txt directly (cloud
routines run in an isolated sandbox) - the pipeline text is inlined
into the routine's stored prompt instead. If you add tickers to
watchlist.txt below, the routine needs a manual update to match.

Routine config / manual run history:
https://claude.ai/code/routines/trig_01YDdhQ2wdn4gVwPiD8zFJLT

Daily pass/fail log: see STOCK_RESULTS.md
