---
name: twenty-enrich
description: "Find what the workspace does not know about a company or a person, and offer it back as field updates."
---

# Enrich Skill

Find what the workspace does not know about a company or a person, and offer it back as field updates.

## Search

Use app_exa_web_search with its category set deliberately. It is preloaded, so call it directly without looking it up first. If it is not available, say the workspace has no web search rather than guessing at answers.

- Set category to "company" for an organisation: what it does, size, funding, headquarters, recent news.
- Set category to "people" for an individual: current role, employer, background.
- Set category to "news" when the user asks what changed rather than who they are.

Start from what the record already holds. A domain makes a far better query than a company name, and a name plus an employer beats a name alone. Run several narrow searches rather than one broad one, and stop as soon as the field you were filling is answered.

## Reconcile

Read the record first and enrich only what is empty or clearly wrong. Overwriting a value a human typed is the fastest way to lose their trust in this feature.

Keep the source for every fact. State plainly which facts came from the web and which were already in the workspace.

## Produce

Show a short table of proposed changes: the field, what it holds now, what you propose, and where it came from. Then ask before writing.

After approval, apply the changes in a single update call and confirm what was written.

Anything you found that has no field to live in belongs in a note on the record, not squeezed into an unrelated field.

## Rules

- Never invent a value to fill a gap. An empty field beats a wrong one.
- Confidence matters: if the search returned a different person who shares the name, say so and stop.
- Do not enrich a large set of records unless the user asked for exactly that, and confirm the scope before you start.
