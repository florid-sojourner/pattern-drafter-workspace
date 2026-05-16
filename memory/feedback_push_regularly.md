---
name: Push commits to origin regularly, don't accumulate local-only commits
description: Justin treats local-only commits as risky; push after each commit (or after a small batch) so work survives a machine loss
type: feedback
originSessionId: 1fd8a42e-4e6f-4889-9943-3738b205d314
---
After committing on a project that has a remote, push promptly. Don't
accumulate many local commits without pushing — Justin treats that as
risky. Default cadence: push after each commit, or after a small batch
of related commits if they're going to land within minutes.

**Why:** Justin stated this directly during the pattern-drafter rebuild
(2026-04-24) after I'd accumulated 15 unpushed commits on `main` over
two days: "why haven't you been committing and pushing? This is so
risky."

**How to apply:**
- After `git commit` on a project with a remote, run `git push` (and
  `-u origin <branch>` the first time on a new branch to set upstream
  tracking).
- Pushing to `main` of a private solo-dev repo doesn't require pre-
  authorization; it's the expected workflow. (For shared / public
  repos, the standard "ask before pushing" rule still applies.)
- If a branch is ahead of `origin` after multiple commits in a session,
  catch up at the end of the relevant unit of work. Don't end a session
  with unpushed commits if the remote exists.
- Force-push and pushes to `main` of shared repos still warrant
  explicit confirmation. This memory is about the routine fast-forward
  push, not destructive operations.
