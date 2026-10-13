---
name: prove-it
description: Print the file name and line numbers of the evidence used to make previous claims. Assumes this is being used in a code repository.
---

For the previous claim made, print the file name and line numbers where the evidence was found for that claim. Be as concise as possible.

When a claim depends on library or platform behavior, prioritize public API documentation via Dash or official documentation. Cite the documentation URL or docset entry and the relevant API contract alongside application file paths and line numbers. Use installed dependency source when documentation is unavailable, ambiguous, or implementation-specific evidence is needed. Locate Ruby gems with `bundle show` or `bundle open`, and Node packages with package resolution such as `require.resolve`. Distinguish judgments from facts supported by evidence.
