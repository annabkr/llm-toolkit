# Google Python Style Guide

## Core Language Rules

### Linting & Code Quality
Make sure you run `pylint` on your code with appropriate suppressions using `# pylint: disable=` comments when warnings are incorrect for your context.

### Import Guidelines
Use full package paths for imports: `from x import y` rather than relative imports. The namespace management convention is simple when module sources are clearly indicated.

**Examples:**
```python
# Good
from package.submodule import function_name
from package import module

# Avoid
from . import sibling_module
```

### Exception Handling
Leverage built-in exceptions like `ValueError` for precondition violations. Avoid bare `except:` statements; instead catch specific exceptions or use broad catching only at isolation points. Keep try/except blocks minimal to prevent masking unexpected errors.

**Examples:**
```python
# Good
try:
    value = collection[key]
except KeyError:
    return default_value

# Bad - too broad
try:
    value = collection[key]
except:
    return default_value
```

### Mutable Global State
Minimize module-level variables that change during execution. When necessary, prepend underscore to indicate internal status and provide explanatory comments about design rationale.

### Comprehensions
Single-clause list/dict comprehensions are acceptable; multiple `for` clauses reduce readability and should be expanded into loops instead.

**Examples:**
```python
# Good
result = [x.upper() for x in items]

# Bad - too complex
result = [(x, y) for x in range(10) for y in range(20) if x != y]

# Better
result = []
for x in range(10):
    for y in range(20):
        if x != y:
            result.append((x, y))
```

## Style Conventions

### Line Length & Formatting
The maximum line length is 80 characters, with exceptions for long URLs, import statements, and flags in comments. Use implicit line joining within parentheses rather than backslash continuation.

**Examples:**
```python
# Good - implicit line joining
foo = long_function_name(
    var_one, var_two,
    var_three, var_four
)

# Avoid backslash continuation
foo = long_function_name(var_one, var_two, \
                        var_three, var_four)
```

### Indentation
Use exactly 4 spaces per indentation level; never use tabs. Align continuation lines either with opening delimiters or use hanging 4-space indents.

**Examples:**
```python
# Good - aligned with opening delimiter
result = function_with_many_arguments(
    argument_one, argument_two,
    argument_three, argument_four
)

# Good - hanging indent
result = function_with_many_arguments(
    argument_one, argument_two,
    argument_three, argument_four
)
```

### Naming Standards

| Type | Naming Convention | Examples |
|------|------------------|----------|
| Modules/packages | `lower_with_under` | `my_module.py`, `data_utils` |
| Classes | `CapWords` | `MyClass`, `DataProcessor` |
| Functions/methods | `lower_with_under()` | `my_function()`, `process_data()` |
| Constants | `CAPS_WITH_UNDER` | `MAX_SIZE`, `DEFAULT_TIMEOUT` |
| Internal items | prepend `_` | `_internal_helper()`, `_cache` |

**Avoid single-letter names** except for:
- Loop counters: `for i in range(10):`
- Exception variables: `except ValueError as e:`
- Established mathematical notation: `x, y, z` in coordinate systems

## Documentation

### Docstrings
Use triple double-quotes (`"""`) for all docstrings. Function docstrings should include one-line summary, then blank line, followed by description with `Args:`, `Returns:`, and `Raises:` sections as applicable.

**Key principle:** A docstring should give enough information to write a call to the function without reading the function's code.

**Examples:**
```python
def fetch_bigtable_rows(big_table, keys, other_silly_variable=None):
    """Fetches rows from a Bigtable.

    Retrieves rows pertaining to the given keys from the Table instance
    represented by big_table. Silly things may happen if other_silly_variable
    is not None.

    Args:
        big_table: An open Bigtable Table instance.
        keys: A sequence of strings representing the key of each table row
            to fetch.
        other_silly_variable: Another optional variable, that has a much
            longer name than the other args, and which does nothing.

    Returns:
        A dict mapping keys to the corresponding table row data
        fetched. Each row is represented as a tuple of strings. For
        example:

        {'Serak': ('Rigel VII', 'Preparer'),
         'Zim': ('Irk', 'Invader'),
         'Lrrr': ('Omicron Persei 8', 'Emperor')}

        If a key from the keys argument is missing from the dictionary,
        then that row was not found in the table.

    Raises:
        IOError: An error occurred accessing the bigtable.Table object.
    """
```

### Comments
Comment tricky code sections thoroughly. Inline comments should be 2+ spaces from code. Avoid describing what code obviously does; instead explain why non-obvious logic exists.

**Examples:**
```python
# Bad - obvious comment
x = x + 1  # Increment x

# Good - explains why
x = x + 1  # Compensate for border width
```

## Type Annotations

Enable type checking tools like `pytype` when feasible. Annotate public APIs and complex code. Use `str | None` syntax over deprecated `Optional`. Prefer abstract types like `collections.abc.Sequence` over concrete `list` types.

Import typing symbols directly: `from typing import Any, cast` rather than `import typing`.

**Examples:**
```python
from typing import Any
from collections.abc import Sequence

def greeting(name: str) -> str:
    return f"Hello {name}"

def process_items(items: Sequence[int]) -> list[int]:
    return [x * 2 for x in items]

def flexible_function(data: Any) -> str | None:
    if not data:
        return None
    return str(data)
```

## Key Practices

### Resource Management
Explicitly close files/sockets using `with` statements:

```python
# Good
with open('file.txt') as f:
    data = f.read()

# Bad
f = open('file.txt')
data = f.read()
f.close()
```

### TODO Comments
Use `TODO` comments linked to bug references:

```python
# TODO(username): Add error handling here (bug #12345)
```

### Function Length
Keep functions under ~40 lines; break up longer functions for readability.

### Mutable Defaults
Never use mutable defaults:

```python
# Bad
def foo(a, b=[]):
    b.append(a)
    return b

# Good
def foo(a, b=None):
    if b is None:
        b = []
    b.append(a)
    return b
```

### Truthiness Checks
Use `if foo:` for implicit falsy checks; explicitly test `if foo is None:` when distinguishing None from other falsy values.

```python
# Good
if not items:
    return

if value is None:
    value = default

# Bad
if len(items) == 0:
    return

if value == None:
    value = default
```

## Philosophy

The guide emphasizes balance between "safety and clarity" versus flexibility, encouraging judgment-based annotation rather than universal coverage. Apply these rules with common sense and prioritize code readability above strict adherence when there's a compelling reason.
