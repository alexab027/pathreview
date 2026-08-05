## Week 7 — Issue selection

**Issue link:** https://github.com/ascherj/pathreview/issues/153

**Issue title:** Faithfulness checker crashes when a context chunk has `text: None`

**Tier:** [x] Tier 1 [ ] Tier 2 [ ] Tier 3

**Problem summary:**  
The faithfulness checker combines text from retrieved context chunks before comparing that context with generated feedback. It currently uses `chunk.get("text", "")`, which handles a missing `text` key but not a key whose value is explicitly `None`. When `None` is passed into `" ".join(...)`, the checker raises a `TypeError` instead of returning a score. A successful fix would normalize unavailable chunk text to an empty string while preserving valid text.

**Why this issue is a good fit:**  
This issue is narrowly scoped to one method in the RAG evaluation code and has a clear reproduction case. The issue also points to an existing failing unit test, so I can verify the bug and confirm the fix without needing to understand the entire application. The expected change appears small and does not involve database schemas, API design, or major architectural changes. This makes it appropriate for a Tier 1 first open-source contribution.

**Branch name:** `fix/153-faithfulness-none-context`

**Setup confirmation:** [x] App runs locally at localhost:5173

**Cohort ledger:** [x] Issue added to cohort ledger

## Week 8 — Reproduction & solution planning

**Reproduction commit link:** https://github.com/alexab027/pathreview/commit/9942e192ca9c1f8ff2441f0c9b1e1aa42db0faf8

**Reproduction summary:**  
I reproduced the issue by calling `FaithfulnessChecker.check()` with a context chunk containing `{"text": None}`. The method raised a `TypeError` because `None` was passed into `" ".join(...)`.

**PLAN.md link:** https://github.com/alexab027/pathreview/blob/fix/153-faithfulness-none-context/PLAN.md

**Blockers or open questions:**  
I am unsure whether the fix should only handle `None` values or also handle other non-string context values.

## Week 9 — Solution building & PR submission

### Check-in 1 (mid-week)

**Current progress:**  
I reproduced the issue, completed `PLAN.md`, and implemented a fix in `FaithfulnessChecker.check()` so context chunks with `text: None` are treated as empty strings instead of causing a `TypeError`.

I also added a regression test covering a `None` context chunk alongside a valid context chunk.

Before implementation, `make test-unit` reported 53 failing tests and 375 passing tests. The relevant failing test, `test_none_context_chunk_text`, reproduced issue #153.

Opened a draft PR and asked for review on slack.

**Next steps:**  
Run the full test suite and code quality checks, confirm that no new failures were introduced, wait for peer/mentor feedback.

**Blockers:**  
The repository has pre-existing unit test, Ruff, and mypy failures unrelated to this issue.

---

### Check-in 2 (end of week)

**PR link:** https://github.com/ascherj/pathreview/pull/618

**Branch:** `fix/153-faithfulness-none-context-pr`

**What you built:**  
Updated `FaithfulnessChecker.check()` so context chunks with `text: None` are normalized to an empty string before the context text is joined. This prevents the existing `TypeError` while preserving normal behavior for valid context text.

I also added a regression test covering a mix of `None` and valid context chunks.

I had to edit my pr to not include the docs, just the actual fix and submitted!

**Tests added or updated:**  
Updated `tests/unit/test_faithfulness_checker.py` with a regression test for `None` context text alongside valid context text.

The targeted `test_none_context_chunk_text` test now passes.

Before the fix, the full unit suite reported 375 passing and 53 failing tests. After the fix, it reported 376 passing and 52 failing tests. No new failures were introduced.

The three remaining failures in `tests/unit/test_faithfulness_checker.py` were already present before my change:

- `test_partial_support_returns_middle_score`
- `test_multiple_context_chunks`
- `test_multiple_claims_varying_support`

**Self-review confirmation:** [x] make check passes [x] make test-unit passes

Both commands still report documented pre-existing repository failures, but comparison before and after the change confirmed that this contribution introduced no new failures.

**Draft PR feedback received from:**
None. I requested but did not receive feedback in slack

## Week 10 — Iteration & reflection

### Reviewer feedback

**Feedback received:** [ ] Yes [x] No — still awaiting review

**Summary of feedback:**  
No reviewer feedback was provided during Summer 2026. I reviewed the open pull request for comments and did not see any maintainer or reviewer feedback to address.

**How you responded:**  
N/A — no feedback was received.

---

### Reflection

**What was harder than you expected?**  
The Git and pull request workflow was harder than the code change itself. The actual fix was small, but I had to manage pre-existing test failures, pre-commit hooks, unrelated changes from project setup, a merge conflict after upstream changed the same test file, and separate my CodePath journal commits from the clean contribution branch. I also had to learn how to distinguish failures caused by my work from failures that already existed in the repository.

**What did you learn about working in a large codebase?**  
I learned that making a small change in someone else’s codebase requires much more than editing one line. I needed to understand the project’s setup process, testing conventions, branch naming rules, commit format, pull request template, and existing code-quality issues. In my own projects, I can change structure or formatting whenever I want. In a shared codebase, keeping the change narrow and making the reviewer’s job easier are important parts of the contribution.

I also learned the value of establishing a baseline before implementation. Running the full unit suite before and after the fix allowed me to show that the suite changed from 375 passing and 53 failing tests to 376 passing and 52 failing tests, with no new failures introduced.

**How did AI tools help — and where did they fall short?**  
AI tools were most useful for explaining the root cause of the bug, helping me interpret terminal errors, suggesting commands, and helping me organize `PLAN.md`, `JOURNAL.md`, and the pull request description.

AI assistance was less reliable when the exact state of my Git repository mattered. Some suggested steps did not initially account for staged versus unstaged files, pre-commit behavior, merge conflicts, or the fact that my journal commits and contribution commits needed to serve different purposes. I still had to inspect `git status`, read the command output carefully, compare the repository before and after my change, and decide which files actually belonged in the pull request.

**What would you do differently if you started over?**  
I would create two branches from the beginning: one CodePath tracking branch containing `JOURNAL.md` and `PLAN.md`, and one clean contribution branch containing only the implementation and test changes. That would have prevented the first pull request from including course documentation.

I would also run the baseline checks before modifying any files, record the results immediately, and inspect the pre-commit configuration earlier. Finally, I would avoid running a formatter across an entire existing test file when I only needed to add one small test, because that created unrelated formatting changes in the diff.

**What are you most proud of from this module?**  
I am most proud that I completed the full open-source contribution process rather than only writing the fix. I reproduced the issue, identified the root cause, wrote a plan, implemented and tested the change, documented pre-existing failures, worked through Git and pre-commit problems, and opened a pull request that another developer could understand and verify.
