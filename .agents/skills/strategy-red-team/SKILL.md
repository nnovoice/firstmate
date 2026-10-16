---
name: strategy-red-team
description: Load when a strategy project scout reports a verdict and before that verdict is relayed, when briefing a red-team scout for that verdict, when reading its report, or when deciding whether a verdict may proceed to a proposal.
user-invocable: false
metadata:
  internal: true
---

# Strategy red team

Every strategy verdict, positive or negative, gets a red-team scout before it is relayed to the captain as a result.
This skill owns the red-team contract for every strategy project: the role contract, the model-family requirement, the common checklist, the report format, the looks ledger, pre-registration, and the proposal gate.
It adds no scripts and no brief flags; firstmate writes the red-team brief by hand from this contract and the ordinary scout scaffold.

## Role contract

- The red team is a scout whose goal is to find the strongest reasons the original verdict is wrong.
- It reads the original report, rules, pre-registration, frozen code, and run read-backs, and may run its own backtests in the same QuantConnect project under the same limits as the original scout: statistics and closed trades only, one run at a time, and no data export.
- The original side answers an open challenge through firstmate, because crewmates never talk to each other directly.
- The brief names the inputs by path, the strategy-specific challenges on top of the common checklist, and any budget.

## Different model family

The red team runs on a different model family from the original scout.
Resolve the family from the original task's recorded dispatch, then pick a verified harness and model outside it through the ordinary `harness-adapters` dispatch path.
A red team on the same family as the original is not a red team; report the gap rather than launching one.

## Common red-team checklist

This checklist applies to every strategy project, and every red-team brief includes it.

- Faithfulness: each implemented rule traced to the source quote or spec line; each approximation named with its expected bias.
- Timing: every signal uses only data known at order time (includes the split-adjusted volume leak in learnings).
- Universe: point-in-time membership, no survivorship; check that coverage filters do not exclude the names the source actually trades.
- Fill realism: stop same-day vs next-day, gap-through fills, market-on-open assumptions, borrow for shorts.
- Cost sensitivity: rerun at half and double the slippage assumption; report the break-even cost.
- Concentration: remove the top five trades; per-year table; flag any result carried by one year or regime.
- Neighbourhood: shift each numeric parameter 20% either way.
- Live parity: every variable in a surviving rule is live-available on QuantConnect with identical values and timing.
- Baseline: same exits with random entries from the same universe, and benchmark buy-and-hold.
- Reproduction: rerun one backtest from the frozen code hash; compare statistics to the read-back.
- Looks audit: pre-registration hash timestamp earlier than the first run; count of out-of-sample reads.
- For a negative: rule-toggle runs on the source's named examples to name the binding rule; does loosening change the sign or only the size of the loss.
- For a positive: bootstrap of the mean trade; the pre-registered kill condition checked honestly.

## Report format

The red-team report is `data/<task-id>/report.md` with one row per challenge.
Each row states the claim, the test, the result, and whether the verdict changes (yes or no).
The report ends with one final line: `verdict stands` or `verdict changes because ...`.
"The verdict stands" is an allowed outcome; a red team that finds nothing verdict-changing has done its job.
When reading the report, treat a row whose verdict-changes column is yes as an open verdict-changing challenge until the original side answers it.

## Looks ledger

Each strategy project keeps a looks ledger committed in the project repo.
It has one line per out-of-sample window and one line for each time a window was read.
A window that has been read is spent for rule changes.
New rules are judged on unspent windows or forward paper trading only.
The red team's own out-of-sample reads are looks, so it records each of them in the ledger, including its own backtests by id.
Because a read window is spent, the red team does not change rules on windows it reads.

## Pre-registration before the first run

Every new hypothesis is pre-registered with a hash before its first run.
The pre-registration states the result that would kill the hypothesis.
A hypothesis whose hash postdates its first run was not pre-registered; the checklist's looks audit reports that as a finding.

## Gate

No `/opsx:propose` of a trading rule until the red-team report exists and no open challenge changes the verdict.
Firstmate checks this gate before dispatching a proposal for a strategy project.
A verdict with no red-team report, or with an open verdict-changing challenge, goes back to the red team or to the captain, not to a proposal.
