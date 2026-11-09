# Test results

All sites, clients and emails are fictional. I wrote the expected result for every checkpoint (`eval_cases.csv`) before the first run.

## Detection test (`SIM Eval`)

`SIM Eval` deletes the test state, then calls the scanner five times in a row, once per simulated day, in a separate `eval` scope. The test state never touches the live state. After day 5 it reads the scan log of the run and compares each site and day with the expected answer.

A checkpoint has one of four expected answers: `none`, `high`, `urgent` or `cannot_check`. A **missed** alert means an alert was expected and none came. A **false alarm** means an alert came when none was expected.

| Run | Checkpoints | Correct | Missed | False alarms | Note |
| --- | --- | --- | --- | --- | --- |
| 2026-11-09 20:28 | 40 | stopped after day 1 | | | test loop bug, see below |
| 2026-11-09 20:29 | 40 | stopped after day 1 | | | same bug, first fix not enough |
| EVAL-20261109-203059 | 40 | 40 | 0 | 0 | |
| EVAL-20261109-203712 | 40 | 40 | 0 | 0 | rerun before the draft tests |
| EVAL-20261109-204030 | 40 | 40 | 0 | 0 | rerun after the second draft fix |

Times are Europe/Warsaw. Per-checkpoint rows of the last run: `test-results.csv`.

**The loop bug.** The eval workflow calls the scanner with an Execute Sub-workflow node inside a loop. On day 1 the scanner's last node was the "Confirmed alerts only" filter, which had nothing to pass on, so the sub-workflow returned no item and the loop stopped. Setting the call to always output an item was not enough on its own: the filter's output still came back on the wrong branch. I added a final `Run finished` node to the scanner that always returns one summary row. The scanner's detection was correct in all three runs; only the test harness was broken.

### What the alerts were

| Day | Alerts |
| --- | --- |
| 1 | none (baseline for 7 sites; site F cannot be checked) |
| 2 | none (C and E pending: change seen once) |
| 3 | C high (OneSignal, only inside GTM); E urgent (our script gone, Pushwoosh added) |
| 4 | B high (PushEngage in the page); D urgent (our script gone, old service worker file still there) |
| 5 | none (already reported, no repeats) |

Site F returned 403 every day and was marked "cannot check" each day without an alert. Site G returned 500 on day 3; that day was skipped and its state kept. Site H mentioned OneSignal and Pushwoosh in a blog post, a `<code>` block, a link and JSON-LD; none of it counted.

## Draft test (`SIM Draft Eval`)

Two parts:

1. Every AI draft for the alerts of the newest detection run must pass the code checks.
2. The checker itself must judge hand-written emails correctly: bad ones flagged, good ones let through.

| Draft run | Alerts | Drafts passing code checks | Checker cases correct | Note |
| --- | --- | --- | --- | --- |
| on EVAL-20261109-203059 | 4 | 4 of 4 | 10 of 10 | my own reading found 2 bad emails |
| on EVAL-20261109-203712 | 4 | 4 of 4 | 12 of 12 | my own reading found 2 bad subjects |
| on EVAL-20261109-204030 | 4 | 4 of 4 | 13 of 13 | my own reading found no problems |

**Fix 1.** The code checks passed, but two technical-help emails thanked the client "for reaching out" and apologised, as if the client had reported the problem. Added: a prompt rule, a check for thanks-for-contacting and apologies, a check that a technical-help email says "it looks like" or "may have", and checker cases K11 and K12.

**Fix 2.** The bodies were now careful, but two subjects still said "Notifications stopped showing". Added: a subject check and a prompt rule for a neutral subject, and checker case K13.

Both fixes came from reading the drafts myself, not from the automated checks. That is the point of the human review step: code checks catch what I already know to look for.

### Checker cases

| Case | Expected | What it tests |
| --- | --- | --- |
| K01 | flagged | names a competitor |
| K02 | flagged | competitor name in lower case |
| K03 | flagged | says we scan the site |
| K04 | flagged | talks about their code |
| K05 | flagged | says we noticed a switch to another provider |
| K06 | flagged | mentions Tag Manager |
| K07 | flagged | asks for a call |
| K08 | flagged | offers free work |
| K09 | passes | a good technical-help email |
| K10 | passes | a good value email |
| K11 | flagged | thanks the client for a report they never sent |
| K12 | flagged | states the problem as a fact |
| K13 | flagged | careful body, but the subject states the problem |

## Limits

- Eight scenarios test the rules as written. Real sites have far more layouts, lazy-loaded scripts and consent banners.
- The drafts were reviewed by hand on 4 alerts per run, not scored automatically.
- The checker finds forbidden words; it cannot judge whether an email is useful. A person still has to read it.
