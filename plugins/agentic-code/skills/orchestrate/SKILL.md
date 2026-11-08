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

Each chunk must be real work: more than a couple of small files, since every implementer first reads the project again. A measured example: two chunks of two small files each cost 2.4 times the tokens and took twice as long as working alone, for the same result. It does not pay for one chunk, for chunks that share files or keep changing each other's interfaces, for exploratory work, or for a change of one or two files. Then work alone.

## 2. Propose, then wait
Show one table: chunk, repository, files it owns, spec tasks, model. Under it, the cost line ("more tokens than doing it alone, roughly one extra project read per chunk; the gain is time and a clean main context") and anything that blocks a clean split. Start only after the user says yes. That yes covers the local work of this run: creating the chunk worktrees and branches, merging chunk branches into the work branch, and removing them after the merge. It does not cover a push, a pull request or a merge into the default branch. If they want proof first, offer the A/B measurement (`improve-workflow`, "Measure a change", single vs orchestrated).

## 3. Work branch and interfaces first
- The **work branch** is the task's feature branch (`git-workflow` §1), never the default branch. Create it before anything else.
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
- The absolute worktree path and the branch (§5).
- What to report (the `implementer` agent defines it).

## 5. Run
- Spawn `agentic-code:implementer` per chunk (Sonnet). A project or user agent with the same purpose (`.claude/agents/`) wins over the plugin's.
- Create every worktree yourself before you spawn, so you know its path and branch, and it starts where you want. Do not use `isolation: "worktree"`: you cannot name its branch in the brief, and its starting point is not yours to choose.
- Put the worktree **inside the repository**, at `<repo>/.claude/worktrees/<chunk>`, so it stays inside the session's working directories. If `.claude/worktrees/` is not git-ignored, tell the implementers to stage files by name only.
- **Same repository:** `git worktree add .claude/worktrees/<chunk> -b <work-branch>-<chunk> <work-branch>`. It starts from the work branch, so it has the interfaces you committed.
- **Another repository:** `git -C <repo> fetch`, then `git -C <repo> worktree add .claude/worktrees/<chunk> -b <feature-branch> origin/<default>`. This chunk branch is that repository's feature branch and becomes its pull request (`git-workflow` §4). If that repository is not a working directory of this session, ask the user to add it (`/add-dir <repo>`) before you start.
- **The check in a worktree:** a subagent's shell may return to your folder after every call, so the brief names the check by its absolute path: `<worktree>/scripts/check.sh`. The `project-setup` template checks the checkout it lives in, whatever the current folder. An older or hand-written script that checks the current folder would check yours instead: then update it from the template first, or give a test command that names the worktree.
- Before you spawn, ask the user to allow the implementers' commands once: `git -C <worktree> …` and `<repo>/.claude/worktrees/*/scripts/check.sh` (for example in `.claude/settings.local.json`). In a headless run, list them in `--allowedTools`.
- **Your check after each merge is the one that counts.** An implementer whose check was refused reports it; you run the check after merging.
- Never the user's own checkout.
- At most three chunks at a time. Independent chunks run in parallel; a chunk that needs another chunk's result waits for it.
- While they run, do not edit the files any chunk owns.

## 6. Review and merge
For each finished chunk, in dependency order:
1. Read the report. Then read the diff, not only the report: `git diff <base>...<branch>`. It must stay inside the chunk's files and meet its criteria.
2. A change outside the chunk, a criterion not met, or a weakened test: send it back with the exact gap. At most two rounds per chunk. Then take it over yourself, or ask the user (`quality-gate`, stop rule). An implementer that failed twice is not started a third time.
3. **Same repository:** merge it into the work branch locally (`git merge --no-ff <chunk-branch>`) and run the check after every merge (`git-workflow` §7). On a conflict, read both sides. If the right answer is not clear, send it back or ask. **Another repository:** do not merge; that branch is reviewed and goes to its own pull request.
4. **No report, or an unfinished one:** keep its worktree and branch, tell the user what is there, and do not merge it.

When all chunks are in, run a fresh `agentic-code:reviewer` (main model, read-only) on the whole diff against the spec (`spec` §5). Fix what it finds that affects correctness or the requirements.

## 7. Close
- Tick the tasks in the spec.
- Report one table: chunk, model, tokens and tool calls (from each completion notice), rounds, check, review findings. Add one line on what the split cost or saved, and say whether it was worth it.
- Remove the worktrees of finished chunks (`git worktree remove`). Delete a same-repository chunk branch with `git branch -d` only once it is merged into the work branch; `-d` refuses otherwise, never bypass it. A branch in another repository stays: it is that repository's pull request. An unmerged chunk stays until the user says otherwise.
- Push and pull request as usual: only with the user's permission, asked separately (`git-workflow`). Several repositories: the PRs refer to each other and are merged together.

## Never
- Start without the user's yes, or split one chunk into one agent per task.
- Let an implementer push, open a PR, merge, rebase or touch files outside its chunk.
- Merge a chunk you have not read the diff of, or skip the check after a merge.
- Run more than three chunks at once, or a third attempt of a failed chunk.
