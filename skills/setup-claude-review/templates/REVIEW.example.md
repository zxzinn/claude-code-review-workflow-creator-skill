# REVIEW.md

<!--
Reference example only — illustrates the level of specificity a good
REVIEW.md needs. Do NOT copy this into a new project verbatim: every
rule below was mined from one specific Django codebase's real bugs and
conventions. A REVIEW.md for a different project must be mined from
that project the same way, not adapted from this file's content.
-->

## Review Language

All review comments MUST be written in English.

## Severity Levels

- **Blocking**: Must fix before merge
- **Nit**: Suggested improvement, not required for merge

## Focus Areas

### Blocking

#### Model `__init__` Deferred Field Safety

- When storing original field values in `__init__` for change tracking, must use `self.__dict__` check:
  ```python
  # CORRECT
  self.original_name = self.name if 'name' in self.__dict__ else None

  # WRONG — causes RecursionError with .only()/.defer()
  self.original_name = self.name
  ```

#### Cross-Object Private Method Calls

- Flag any call to a `_`-prefixed method on an object other than `self` or `cls` (e.g. `service._do_thing()`)
- The `_` prefix means "internal implementation detail"; calling it from outside the owning class breaks encapsulation
- `self._method()`, `cls._method()`, and `super()._method()` are fine

### Nit

#### Over-Engineering

- Flag unnecessary abstractions for single-use cases
- Flag defensive programming for impossible scenarios
- Flag "future-proof" patterns without current requirements

## Skip (Do NOT Review)

- Code formatting and style (handled by linters and pre-commit)
- Migration files whose operations are exclusively auto-generated schema ops with a matching models.py diff
