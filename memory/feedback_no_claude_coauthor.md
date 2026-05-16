---
name: Never add Co-Authored-By Claude trailer to commits
description: Justin commits as himself; do not add the "Co-Authored-By: Claude ..." line that the default git workflow suggests
type: feedback
originSessionId: 1fd8a42e-4e6f-4889-9943-3738b205d314
---
When creating commits on Justin's projects, do NOT append the
`Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>`
trailer (or any variant) to the commit message. Author the commit as
Justin himself — his configured `user.name` / `user.email` already do
this correctly.

**Why:** Justin stated this directly during the pattern-drafter rebuild
(2026-04-24): "always commit as me, not claude." The default workflow
suggests the trailer; he doesn't want it.

**How to apply:** When using HEREDOC-based commit messages, omit the
Co-Authored-By line entirely. Don't substitute a different attribution
trailer either — the commit author identity already credits him.
