# GitHub Integration Best Practices

## Critical Workflow Rule: Push First, Explain Second

**Problem:** Perplexity's response generation has a per-turn output token limit. When a long explanation is followed by a GitHub push tool call, the response can be truncated before the tool call executes — the push never happens.

**Symptom:** Response ends mid-sentence with something like "Pushing now..." but nothing is pushed.

**Fix:** Always structure responses in this order:
1. Fetch the current file SHA (if updating an existing file)
2. Execute the push tool call immediately
3. Then explain what changed (diff only, not full code)

This guarantees the push completes even if the explanation gets truncated.

---

## SHA Management

- Always fetch the current SHA before updating a file using `get_file_contents`
- The SHA changes on every commit — never reuse a SHA from a previous turn
- If a push returns a SHA mismatch error, re-fetch and retry immediately
- Use `list_commits` to verify whether a push actually happened before retrying

---

## Verifying Push Success

If unsure whether a push happened (e.g. after a truncated response):
1. Call `list_commits` on the target branch with `perPage: 3`
2. Check the latest commit message against the expected change
3. Only re-push if the commit is absent

---

## Response Length Management

- Never display full file contents in the response — push to GitHub instead
- Only show diffs or key changed sections in the response text
- Break large changes into smaller separate asks if possible (e.g. "fix bug 1" then "fix bug 2")
- If a response is cut off, the user can ask "check if it was pushed" as a recovery step

---

## Push Tool Parameters

- `push_files` for multi-file commits in one shot (preferred during low load timeframes - 3am - 8pm UTC+8)
- `create_or_update_file` for single-file updates (requires SHA) - (preferred during high load timeframes - 8pm - 3am UTC+8)
- If 'push_files' fail the first time, always use 'create_or_update_file' for that session going forward.
- Always push to master branch. Never 'main' branch which is deprecated.
- Smallest → largest — sequence pushes by file size ascending
- Partial edits preferred — if only a few lines change, describe the diff rather than rewriting the full file, to conserve context
- Always include a meaningful `message` following conventional commits format:
  - `feat:` new feature
  - `fix:` bug fix
  - `perf:` performance improvement
  - `chore:` maintenance
  - `refactor:` code restructure without behaviour change
