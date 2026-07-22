## Solution plan

**Issue:** Faithfulness checker crashes when a context chunk has `text: None` — [issue link]

### Understand

`FaithfulnessChecker.check()` joins the text from each context chunk.

When a chunk contains:

```python
{"text": None}
```

`chunk.get("text", "")` returns `None` because the `"text"` key exists. Passing `None` into `" ".join(...)` raises a `TypeError`.

Expected behavior: a context chunk with `None` text should be handled without crashing.

Actual behavior: the checker raises:

```text
TypeError: sequence item 0: expected str instance, NoneType found
```

### Map

Files and functions involved:

- `rag/evaluator/faithfulness_checker.py`
  - `FaithfulnessChecker.check()`
- `tests/unit/test_faithfulness_checker.py`
  - `test_none_context_chunk_text`

Files I expect to touch:

- `rag/evaluator/faithfulness_checker.py`
- `tests/unit/test_faithfulness_checker.py`, if more test coverage is needed

### Plan

1. Review the existing context-building logic and related tests.
2. Update the context-building logic so missing or `None` text is treated as an empty string.
3. Run `test_none_context_chunk_text` to verify the bug is fixed.
4. Run the full faithfulness checker test file.
5. Run the broader test suite to check for regressions.

### Inputs & outputs

Input:

```python
FaithfulnessChecker().check(
    "Knows Python.",
    [{"text": None}]
)
```

Expected behavior:

- The method does not raise a `TypeError`.
- `None` text is treated as empty text.
- Valid string context continues to work normally.

### Risks & unknowns

- Treating `None` as empty text could result in an entirely empty context, so I need to check how `FaithfulnessChecker.check()` handles that later in the method.
- I am unsure whether non-string values besides `None` should also be handled.
- I need to verify whether the existing test fully covers mixed valid and `None` context chunks.

### Edge cases

- `{"text": None}`
- A chunk with no `"text"` key
- `{"text": ""}`
- An empty context list
- A mixture of valid text and `None`
