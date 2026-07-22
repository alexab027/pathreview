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

**Reproduction commit link:** [add commit link after pushing]

**Reproduction summary:**  
I reproduced the issue by calling `FaithfulnessChecker.check()` with a context chunk containing `{"text": None}`. The method raised a `TypeError` because `None` was passed into `" ".join(...)`.

**PLAN.md link:** [add PLAN.md GitHub link after pushing]

**Walkthrough video (recommended):**

**Blockers or open questions:**  
I am unsure whether the fix should only handle `None` values or also handle other non-string context values.
