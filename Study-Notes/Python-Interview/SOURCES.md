# Resources and references: Python Interview

Accessed 19 Nov 2026. Status: Cross-verified / Single source / Not read.

| Title | Link | Used for | Status |
|---|---|---|---|
| Python Language Reference: Data model | https://docs.python.org/3/reference/datamodel.html | Mutability, tuple holding a mutable object, `__hash__`/`__eq__` contract, C3 MRO mention | Opened; first 100 KB only; the text on user-defined class defaults was cut off, so "defining `__eq__` sets `__hash__` to None" is **Ex.** |
| Python docs: Glossary | https://docs.python.org/3/glossary.html | GIL (including free-threaded build since 3.13), generators, decorators | Opened |
| Python docs: The Python 2.3 Method Resolution Order | https://docs.python.org/3/howto/mro.html | C3 linearization properties and the `TypeError` when no consistent order exists | Opened earlier this session |
| Python docs: Time complexity of operations on built-in types | https://docs.python.org/3/builtins/time-complexity.html | list/dict/set costs and caveats | Opened earlier this session |
| Python docs: Sorting Techniques (HOWTO) | https://docs.python.org/3/howto/sorting.html | Sorts are guaranteed stable; Timsort | Opened earlier this session |

Not read / not verified: the Python docs on `asyncio`, `functools`, `dataclasses`, `itertools`, `typing`, and the garbage collector; those details are **Ex.** in the notes. The cyclic GC and reference-counting description is standard knowledge, not checked against the docs here.

Interview question lists (unofficial) were used earlier in the session for the 150-question bank (see `QuestionBank/` in the CS-Fundamentals-Notes repo); no company-specific Python questions are claimed.
