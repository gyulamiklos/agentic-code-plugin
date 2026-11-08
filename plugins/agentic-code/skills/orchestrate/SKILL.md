---
name: orchestrate
description: Run a large spec as an orchestrator - split it into a few independent work chunks (or one chunk per repository), fix the interfaces between them first, let one implementer subagent on a cheaper model build each chunk in its own git worktree, then review, merge and check the result. Use when a spec has two or more independent chunks, when a change spans several repositories, or when the user invokes /orchestrate or asks for parallel or multi-agent work. Only proposes; starts after the user agrees. Not for one chunk, tightly coupled work or a change of one or two files.
argument-hint: "<spec path>"
---

# Orchestrate: one lead, one implementer per chunk

The main session is the lead. It plans, fixes the interfaces, briefs, reviews, merges and checks. Implementer subagents on a cheaper model each build one chunk in their own worktree. The gain is wall-clock time and a main context that stays clean. The cost is more tokens than doing the work alone: every implementer reads the project again. Say this every time you propose it.

## 1. When it pays
All of these:
- There is a spec (`spec`). No spec, no orchestration: write the spec first.
- There are at least two **chunks**. A chunk is a group of related spec tasks, not a single task. It can be committed and tested on its own, it touches files no other chunk touches, and it meets the other chunks only through interfaces the spec fixes.
- **Or** the change spans several repositories (`git-workflow` §4): one chunk per repository.

It does not pay for one chunk, for chunks that share files or keep changing each other's interfaces, for exploratory work, or for a change of one or two files. Then work alone.

## 2. Propose, then wait
Show one table: chunk, repository, files it owns, spec tasks, model. Under it, the cost line ("more tokens than doing it alone, roughly one extra project read per chunk; the gain is time and a clean main context") and anything that blocks a clean split. Start only after the user says yes. If they want proof first, offer the A/B measurement (`improve-workflow`, "Measure a change", single vs orchestrated).

## 3. Interfaces first
- Every interface between chunks (a function signature, a type, an API contract, a file format, a database field) is written in the spec. Or you commit it yourself on the work branch before any chunk starts: the types, a stub, the contract document.
- Anything outside those interfaces that two chunks would both need is not split. Do it yourself first, or make it one chunk.

## 4. Brief each chunk
Each brief stands alone. The implementer does not see this conversation.
- The subagent brief block (`docs/agent-brief.md`) if the project has one, at the top.
- The spec path and the chunk's tasks, by their numbers in the spec.
- The files it owns, and that every other file is off limits.
- The fixed interfaces, verbatim.
- The acceptance criteria for this chunk.
- The check command, and the base commit.
- The worktree path and branch (see below), and what to report (the `implementer` agent defines it).

## 5. Run
- Spawn `agentic-code:implementer` per chunk (Sonnet). A project or user agent with the same purpose (`.claude/agents/`) wins over the plugin's.
- **Same repository:** spawn with `isolation: "worktree"`.
- **Another repository:** create the worktree yourself first, `git -C <repo> worktree add <scratchpad>/<chunk> -b <branch> <base>`, and pass that path. Never the user's own checkout.
- At most three chunks at a time. Independent chunks run in parallel; a chunk that needs another chunk's result waits for it.
- While they run, do not edit the files any chunk owns.

## 6. Review and merge
For each finished chunk, in dependency order:
1. Read the report. Then read the diff, not only the report: `git diff <base>...<branch>`. It must stay inside the chunk's files and meet its criteria.
2. A change outside the chunk, a criterion not met, or a weakened test: send it back with the exact gap. At most two rounds per chunk. Then take it over yourself, or ask the user (`quality-gate`, stop rule). An implementer that failed twice is not started a third time.
3. Merge it into the work branch locally and run the check after every merge (`git-workflow` §7). On a conflict, read both sides. If the right answer is not clear, send it back or ask.

When all chunks are in, run a fresh `agentic-code:reviewer` (main model, read-only) on the whole diff against the spec (`spec` §5). Fix what it finds that affects correctness or the requirements.

## 7. Close
- Tick the tasks in the spec.
- Report one table: chunk, model, tokens and tool calls (from each completion notice), rounds, check, review findings. Add one line on what the split cost or saved, and say whether it was worth it.
- Remove the worktrees and branches of merged chunks (`git worktree remove`, `git branch -d`). An unmerged one stays until the user says otherwise.
- Push and pull request as usual: only with the user's permission, asked separately (`git-workflow`). Several repositories: the PRs refer to each other and are merged together.

## Never
- Start without the user's yes, or split one chunk into one agent per task.
- Let an implementer push, open a PR, merge, rebase or touch files outside its chunk.
- Merge a chunk you have not read the diff of, or skip the check after a merge.
- Run more than three chunks at once, or a third attempt of a failed chunk.
