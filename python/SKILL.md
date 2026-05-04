## Type Annotations

All methods and functions must have return type annotations. Use `from __future__ import annotations` for forward references.

```python
def get_user(user_id: int) -> User | None:
    ...
```

Prefer `X | Y` union syntax (Python 3.10+) over `Optional[X]` or `Union[X, Y]`.

## Docblocks

Public methods, functions, and reusable utilities require docblocks. Simple one-liners and test cases are exempt.

```python
def my_method(self, value: int) -> str:
    """
    Brief one-line description.

    Args:
        value (int): Description.

    Returns:
        str: Description.

    Raises:
        ValueError: If value is negative.

    Example:
        result = service.my_method(42)
    """
```

Include `Example` only when usage is non-obvious or the method is a reusable utility.

## Idiomatic Patterns

**Comprehensions over loops**
```python
squares = [x**2 for x in range(10)]
active_ids = {u.id for u in users if u.is_active}
```

**Unpacking**
```python
first, *rest = items
a, b = b, a
```

**enumerate and zip over manual indexing**
```python
for i, item in enumerate(items): ...
for user, role in zip(users, roles): ...
```

**f-strings over format() or %**
```python
label = f"Hello, {user.first_name}!"
```

**Walrus operator for assign-and-test**
```python
if match := pattern.search(text):
    print(match.group())
```

**EAFP over LBYL**
```python
# good
try:
    value = data["key"]
except KeyError:
    value = default

# avoid
if "key" in data:
    value = data["key"]
```

**Properties over getters/setters**
```python
@property
def full_name(self) -> str:
    return f"{self.first_name} {self.last_name}"
```

**dataclasses for data containers**
```python
from dataclasses import dataclass

@dataclass
class Point:
    x: float
    y: float
```

**collections module for grouping and counting**
```python
from collections import defaultdict, Counter

word_count = Counter(words)
grouped = defaultdict(list)
```

**Generator expressions for lazy evaluation**
```python
total = sum(x**2 for x in range(1000))
```

**pathlib over os.path**
```python
from pathlib import Path

config = Path("config") / "settings.json"
```

**_ for throwaway variables**
```python
for _ in range(5):
    do_something()
```

**Always define __repr__ on classes** for unambiguous debug output.

## Standard Library First

Reach for the standard library before third-party packages:
- `itertools` — iteration patterns (chain, groupby, islice)
- `functools` — lru_cache, partial, reduce
- `contextlib` — custom context managers
- `dataclasses` — value objects
- `pathlib` — file paths
- `typing` — type hints

## What to Avoid

- Mutable default arguments (`def f(items=[])`)
- Bare `except:` or `except Exception` without specific handling or re-raise
- `type(x) == SomeType` — use `isinstance(x, SomeType)` instead
- Manual index tracking — use `enumerate`
- Implicit string concatenation — use f-strings or `textwrap.dedent`
