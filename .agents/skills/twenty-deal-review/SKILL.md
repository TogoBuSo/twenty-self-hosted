---
name: twenty-deal-review
description: "Judge the health of one opportunity or a set of them, and say what to do next."
---

# Deal Review Skill

Judge the health of one opportunity or a set of them, and say what to do next.

## Gather

For each opportunity in scope, read the record and what is linked to it: the company, the people involved, notes, tasks, and recent messages and calendar events. Use group_by_opportunities when the question is about the shape of the pipeline rather than about individual deals.

## Signals

Work through these for every deal. Each one you can answer becomes a line in the output:

- **Momentum.** How long has it sat in its current stage, and how does that compare with the rest of the pipeline?
- **Contact.** When did anyone last exchange a message or meet? Silence against an approaching close date is the strongest warning sign there is.
- **Threading.** How many distinct people at that company have been in contact? A deal running through one person is a deal that dies when that person leaves.
- **Hygiene.** Is the close date set, already past, or implausible? Is the amount missing?
- **Commitments.** Are there open or overdue tasks, and who owns them?

## Produce

For a single deal, write a short verdict, the evidence behind it, and one concrete next step.

For several, lead with the deals that need attention, one line each, then summarise the rest. Sort by risk rather than by size, and reference every opportunity so the user can open it.

Say which signals you could not evaluate and why. A review that hides its gaps is worse than a short one.

## Rules

- This skill only reads. Propose the next step and let the user ask you to create the task.
- Never call a deal at risk without naming the signal that says so.
- Never guess at an amount or a date that is missing. Report it as missing.
