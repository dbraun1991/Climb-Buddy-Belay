# 0005. Fixed kg-threshold model for belay-safety classification

Date: 2026-08-24

## Status
Accepted

## Context
The tool needs a rule for "can person A safely belay person B while B
leads." Real-world belay safety depends on many variables: rope diameter
and friction, belay device and whether it has assisted braking, rope
drag/path, ground conditions, and technique. Modeling all of that would
require inputs the tool doesn't collect and turn a quick-check tool into a
much bigger one.

The README documents the domain simplification directly: in a
low-friction environment (straight rope path indoors), the optimal weight
difference is around ±5 kg, and heavier rope increases the compensation
needed on most devices relative to thin rope.

## Decision
Belay safety is classified purely from the weight difference
(`climber.kg - belayer.kg`) against four fixed bands, implemented in
`classify()` (`index.html:591`):

| Difference | Rating |
|---|---|
| ≤ 10 kg heavier | OK (green) |
| 10–15 kg heavier | Warn (yellow) |
| 15–20 kg heavier | Critical (orange) |
| \> 20 kg heavier | Danger (red) |

At ≥15 kg difference, the UI additionally surfaces a static hint about
assisted-braking devices (Mammut Assist, Edelrid Ohmega, Raed Zead) and
rope-diameter effects, rather than trying to model those factors
numerically.

## Consequences
- The tool gives a fast, understandable first-pass check using only two
  numbers per person (name, weight) — no device, rope, or setup
  questionnaire.
- The thresholds are a simplification, not a certified safety
  calculation, and only account for weight and (qualitatively) rope/device
  via a static text hint. This must stay visible to users as guidance, not
  a substitute for training or manufacturer/device-specific guidance — the
  UI frames results as recommendations ("empfohlen"/"nicht empfohlen"),
  which should be preserved in any future wording changes.
- The bands and hint copy are hardcoded in `classify()`, `SYM`, and the
  gear-hint markup. Changing the model (e.g. adding rope diameter as an
  input, or tuning the band boundaries based on new guidance) means
  editing these in lockstep with the legend in the sidebar and the
  README's documented table — they are currently three separate places
  that must agree.
