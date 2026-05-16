# PEP 8: Python Style Guide

## Code Layout

### Indentation & Line Breaks
- **Use 4 spaces per indentation level** (never tabs)
- Use hanging indents or vertical alignment for continuation lines
- No arguments permitted on first line when not using vertical alignment

**Examples:**
```python
# Good - aligned with opening delimiter
result = function_name(argument_one, argument_two,
                      argument_three, argument_four)

# Good - hanging indent
result = function_name(
    argument_one, argument_two,
    argument_three, argument_four
)

# Bad - arguments on first line without vertical alignment
result = function_name(argument_one, argument_two,
    argument_three, argument_four)
```

### Line Length
- **Maximum 79 characters for code**
- **Maximum 72 characters for docstrings and comments**
- Teams may extend to 99 characters with mutual agreement, keeping comments at 72

### Binary Operators
Breaking convention has shifted from after to before operators. This follows mathematical tradition and improves readability by keeping operators closer to operands.

**Examples:**
```python
# Good - operator at beginning of line
income = (gross_wages
          + taxable_interest
          + (dividends - qualified_dividends)
          - ira_deduction
          - student_loan_interest)

# Avoid - operator at end of line
income = (gross_wages +
          taxable_interest +
          (dividends - qualified_dividends) -
          ira_deduction -
          student_loan_interest)
```

### Blank Lines
- **Two blank lines** surround top-level function and class definitions
- **Single blank line** between methods within classes
- Use sparingly within functions for logical sections

**Examples:**
```python
import os
import sys


def function_one():
    pass


def function_two():
    pass


class MyClass:
    def method_one(self):
        pass

    def method_two(self):
        pass
```

### Imports
- Place at file top after module docstrings
- Group in order: standard library, third-party, local application
- Separate groups with blank lines
- **Absolute imports are recommended**

**Examples:**
```python
# Good
import os
import sys

import requests
from flask import Flask

from mypackage import mymodule
from mypackage.utils import helper

# Bad - mixed grouping
import os
import requests
import sys
from mypackage import mymodule
```

## String Quotes

No preference between single and double quotes, but **maintain consistency**. For triple-quoted strings, always use double quotes per docstring convention.

```python
# Both acceptable - pick one and be consistent
message = "Hello, world"
message = 'Hello, world'

# Docstrings always use double quotes
def my_function():
    """This is a docstring."""
    pass
```

## Whitespace Guidelines

### Avoid extraneous whitespace:
- Inside parentheses, brackets, braces
- Before commas, semicolons, colons
- Between function name and opening parenthesis
- Before indexing/slicing brackets

**Examples:**
```python
# Good
spam(ham[1], {eggs: 2})
if x == 4: print(x, y); x, y = y, x
dct['key'] = lst[index]

# Bad
spam( ham[ 1 ], { eggs: 2 } )
if x == 4 : print(x , y) ; x , y = y , x
dct ['key'] = lst [index]
```

### Required spacing:
- Single space around binary operators (assignment, comparison, Boolean)
- Spaces around `->` in function annotations
- No spaces around `=` for keyword arguments or default values (unless annotated)

**Examples:**
```python
# Good
x = 1
y = 2
long_variable = 3
i = i + 1
submitted += 1
x = x*2 - 1
hypot2 = x*x + y*y
c = (a+b) * (a-b)

def function(default=None):
    pass

def annotated(value: int = 0):
    pass

# Bad
x             = 1
y             = 2
long_variable = 3
i=i+1
submitted +=1
x = x * 2 - 1
hypot2 = x * x + y * y
c = (a + b) * (a - b)

def function(default = None):
    pass
```

## Naming Conventions

| Entity | Convention | Examples |
|--------|-----------|----------|
| Modules/Packages | lowercase with underscores | `my_module`, `data_utils` |
| Classes | CapWords | `MyClass`, `HTTPServer` |
| Functions/Variables | lowercase_with_underscores | `my_function`, `local_variable` |
| Constants | UPPER_CASE_WITH_UNDERSCORES | `MAX_SIZE`, `DEFAULT_TIMEOUT` |
| Private methods/attributes | _single_leading_underscore | `_internal_method`, `_cache` |
| Name mangling (subclasses) | __double_leading_underscore | `__private_attr` |

### Names to avoid
Single characters `l`, `O`, or `I` as variables due to font ambiguity with numerals 1 and 0.

### Class Naming
- Exception names should end in "Error": `ValueError`, `ConnectionError`
- Use CapWords for class names, but lowercase for built-in names that are used like functions

**Examples:**
```python
class CustomerDataProcessor:
    """Good class name."""
    pass

class ValidationError(Exception):
    """Good exception name."""
    pass

MAX_RETRY_COUNT = 3

def calculate_total_price():
    """Good function name."""
    pass

_internal_cache = {}
```

## Comments

### General Guidelines
- **"Comments that contradict the code are worse than no comments"**
- Use complete sentences with proper capitalization
- Block comments indent with code; each line starts with `# ` and single space
- Inline comments need two spaces minimum before the `#`
- Docstrings follow PEP 257 conventions

**Examples:**
```python
# This is a block comment explaining
# the following section of code.
x = x + 1

y = y + 1  # Inline comment with two spaces before

# Bad inline comment
z = z + 1 # Only one space before
```

## Programming Recommendations

### Exception Handling
- Derive from `Exception` rather than `BaseException`
- Catch specific exceptions; avoid bare `except:` clauses
- Use `except Exception:` for broad error catching

**Examples:**
```python
# Good
try:
    value = process_data()
except ValueError as e:
    logger.error(f"Invalid value: {e}")
except KeyError:
    value = default_value

# Bad
try:
    value = process_data()
except:
    value = default_value
```

### Comparisons
- Use `is`/`is not` for None comparisons, never equality operators
- Apply `isinstance()` for type checking, not direct type comparison
- Avoid boolean equality: use `if greeting:` not `if greeting == True:`

**Examples:**
```python
# Good
if value is None:
    return default

if isinstance(obj, str):
    process_string(obj)

if not items:
    return empty_result

# Bad
if value == None:
    return default

if type(obj) == str:
    process_string(obj)

if items == []:
    return empty_result

if greeting == True:
    print("Hello")
```

### Other Practices
- Use `def` statements instead of lambda assignment
- Check sequences by truthiness, not `len()`: `if not seq:` preferred
- Use `.startswith()` and `.endswith()` instead of slicing
- Keep `try` clauses minimal to avoid masking bugs
- Use `with` statements for resource management

**Examples:**
```python
# Good
if text.startswith('prefix'):
    handle_prefix()

with open('file.txt') as f:
    data = f.read()

# Bad
if text[:6] == 'prefix':
    handle_prefix()

f = open('file.txt')
data = f.read()
f.close()
```

## Function & Variable Annotations

PEP 8 recommends following PEP 484 syntax. Type checkers remain optional; Python interpreters ignore annotations by default. Stub files (.pyi) distribute type information separately when needed.

**Examples:**
```python
def greeting(name: str) -> str:
    return f"Hello {name}"

def process(items: list[int]) -> None:
    for item in items:
        print(item)

count: int = 0
names: list[str] = []
```

## A Foolish Consistency is the Hobgoblin of Little Minds

PEP 8 emphasizes that these are guidelines, not absolute rules. Know when to be inconsistent:

1. When applying the guideline would make the code less readable
2. To be consistent with surrounding code that also breaks the guideline
3. When the code predates the guideline
4. When compatibility with older Python versions is required

**"A style guide is about consistency. Consistency with this style guide is important. Consistency within a project is more important. Consistency within one module or function is the most important."**
