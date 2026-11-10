---
name: commit-writing
description: Use whenever writing text that goes into a repository or onto GitHub - commit messages, PR titles and descriptions, code comments, docstrings, READMEs, changelogs, release notes, issue and review comments. Triggers include "commit", "kommitolj", "pusholj", "PR", "pull request", "nyiss PR-t", "changelog", "README", "release notes", "kódkomment". Project rules win where they exist; this skill covers how the text reads.
---

# Commit and code writing

Text in a repository is read later, mostly by coding agents (Claude sessions,
reviewers, the next agent picking up the work) and sometimes by a person in a
hurry. Completeness and clarity are the tone: plain words, active verbs, one
idea per sentence.

<audience>
Write for an agent that has only this text and the diff, and must act
correctly on it: review it, build on it, debug it months later, or decide
whether it caused a regression. So:
- State every behaviour change with its exact values: status codes, defaults,
  limits, timeouts, worst cases, renamed fields, changed messages.
- Name what callers or other code must change, and what stays compatible.
- Name the root cause, not only the symptom.
- Structure beats flow: headings, bullets with exact values, a table when
  several values share the same attributes. Each bullet is still a full,
  plain sentence or a label with its explanation, so a person can read it too.
- Completeness never means padding: no repeated statements, no narrative of
  the debugging session, no adjectives about quality.
</audience>

<precedence>
Project rules come first. Before writing, check in this order and follow what
you find: the repo's CLAUDE.md and any project skill, a CONTRIBUTING file or
PR template, then `git log --oneline -15` to match the existing convention
(Conventional Commits or not, language, capitalisation, scope prefixes). Where
none of these settle the format, `git-workflow` does (§3: conventional prefix,
lowercase after it). This skill only fills the gaps and decides how the text
reads. Attribution lines follow `git-workflow` too: none, unless the project
asks for them.
</precedence>

<commit_messages>
- Subject line: imperative mood, about 50 characters, at most 72, no full stop.
  It names the change from the outside: "fix: crash when a garden has no zones",
  not "Updated GardenView.dart" or "Fixes".
- Language: the repo's existing convention; if there is none, English.
- Body, when the change is not obvious from the subject: a blank line, then
  wrapped prose at about 72 characters explaining why the change was needed
  and what it does about it. The diff already shows how.
- Mention what a reviewer would otherwise trip over: a behaviour change, a
  migration, a deliberate limitation, a follow-up left open.
- One logical change per commit. If the subject needs "and", it is probably two.
- Leave out: "This commit...", file-by-file narration, the story of the
  debugging session, emojis unless the repo uses them.
</commit_messages>

<pull_requests>
- Title: same rules as a commit subject.
- Body under these headings, each kept as short as its content allows:
  `Problem` (symptom, who hit it, root cause);
  `Change` (what the code does now, with exact values; a table when several
  values share attributes);
  `Compatibility` (what callers must change, what stays the same, any changed
  message, field or default; "None" when nothing changes);
  `Testing` (tests added and what each proves, commands run, what was checked
  by hand, what was not tested);
  `Not in this PR` (deliberate limits and follow-ups), only when there are any.
- Link the issue or ticket. Screenshots or recordings for UI changes.
- No marketing tone, no "This PR introduces a robust and seamless...".
</pull_requests>

<code_comments>
- A comment explains why, or a constraint the code cannot show: a workaround
  for a platform bug, a business rule, an ordering that matters, a unit.
- It does not repeat what the next line does, and it does not tell history
  ("changed this because the old version..."); history belongs in the commit.
- Docstrings say what a function promises: inputs, output, errors, side
  effects, in the language's usual docstring form.
- TODOs name the condition for doing them: "TODO: remove once iOS 17 is dropped".
</code_comments>

<docs_and_changelogs>
- READMEs open with what the thing is and how to run it, in that order.
  Commands go in code blocks that can be copied as they are.
- Changelog and release notes are written for users: what they will notice,
  grouped as Added, Changed, Fixed, Removed. Internal refactors only if they
  affect users.
</docs_and_changelogs>

<writing_rules>
These shape the text as it is written; nothing is left for a later pass.
- Every behaviour is written with its exact value in the same sentence:
  "Retries 429, 502, 503 and 504, up to 3 times (4 attempts)."
- Each fact is written once, in the section where a reader looks for it.
- Verbs say what the code does ("converts", "retries", "drops", "rejects").
  Quality adjectives stay out: the numbers carry the improvement.
- The subject line is written to read clearly in a `git log --oneline` of 200
  lines.
- Only what the diff does goes in; plans and guesses go under "Not in this PR".
</writing_rules>

<examples>
Weak:  "Updated files for the zone feature and fixed some bugs"
Good:  "fix: hide empty zones from the garden overview"

       Gardens created before the zone migration have a placeholder zone
       with no plants, which showed up as a blank card. Filter zones with
       no plants out of the overview; the zone itself is kept so the
       migration can still fill it later.

Weak comment:  // loop over the items
Good comment:  // The API returns items newest-first; the chart needs oldest-first.
</examples>
