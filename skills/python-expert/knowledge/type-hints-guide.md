# PEP 484: Type Hints - Practical Usage Guide

## Overview

PEP 484 introduces type hints using function annotations. **Importantly: no type checking happens at runtime**—type hints are intended for static analysis tools like mypy, pyright, and pyre.

## Basic Types

The simplest form annotates arguments and return values with built-in classes:

```python
def greeting(name: str) -> str:
    return 'Hello ' + name

def add(a: int, b: int) -> int:
    return a + b

def is_valid(value: float) -> bool:
    return value > 0.0
```

Type hints work with:
- Built-in classes (`str`, `int`, `float`, `bool`, `bytes`)
- Standard library types
- Abstract base classes
- User-defined classes

## Generics and Collections

For containers with specific element types, use generic syntax with square brackets:

```python
from typing import List, Dict, Set, Tuple

def process(items: List[int]) -> None:
    for item in items:
        print(item * 2)

def lookup(data: Dict[str, str]) -> None:
    for key, value in data.items():
        print(f"{key}: {value}")

def unique_items(items: Set[str]) -> List[str]:
    return sorted(items)

def get_coordinates() -> Tuple[float, float]:
    return (42.3601, -71.0589)
```

### Modern Python 3.9+ Syntax

Python 3.9+ allows using built-in collection types directly:

```python
def process(items: list[int]) -> None:
    pass

def lookup(data: dict[str, str]) -> None:
    pass

def coordinates() -> tuple[float, float]:
    return (0.0, 0.0)
```

### Abstract Collection Types

**Prefer abstract collection types** for function parameters rather than concrete types for better flexibility:

```python
from collections.abc import Sequence, Mapping, Iterable

# Good - accepts any sequence type
def sum_items(items: Sequence[int]) -> int:
    return sum(items)

# More restrictive - only accepts lists
def sum_items(items: List[int]) -> int:
    return sum(items)

# Good - accepts any mapping
def process_config(config: Mapping[str, str]) -> None:
    pass

# Good - accepts any iterable
def process_stream(data: Iterable[str]) -> None:
    for item in data:
        print(item)
```

## TypeVar and Generic Functions

Create reusable generic functions with `TypeVar`:

```python
from typing import TypeVar, Sequence

T = TypeVar('T')

def first(items: Sequence[T]) -> T:
    """Get the first item, preserving type."""
    return items[0]

def last(items: Sequence[T]) -> T:
    """Get the last item, preserving type."""
    return items[-1]

# Usage preserves types
numbers: list[int] = [1, 2, 3]
first_num: int = first(numbers)  # Type checker knows this is int

names: list[str] = ["Alice", "Bob"]
first_name: str = first(names)  # Type checker knows this is str
```

### Constrained Type Variables

Constrain type variables to specific types:

```python
from typing import TypeVar

AnyStr = TypeVar('AnyStr', str, bytes)

def concat(x: AnyStr, y: AnyStr) -> AnyStr:
    """Concatenate strings or bytes, but not mixed."""
    return x + y

# Valid
result1 = concat("hello", "world")  # str + str -> str
result2 = concat(b"hello", b"world")  # bytes + bytes -> bytes

# Invalid (type checker will complain)
result3 = concat("hello", b"world")  # str + bytes -> error
```

### Bounded Type Variables

Restrict type variables to subclasses:

```python
from typing import TypeVar

class Animal:
    def make_sound(self) -> str:
        return "..."

class Dog(Animal):
    def make_sound(self) -> str:
        return "Woof!"

AnimalType = TypeVar('AnimalType', bound=Animal)

def get_sound(animal: AnimalType) -> str:
    """Works with Animal or any subclass."""
    return animal.make_sound()
```

## Union and Optional Types

Indicate multiple acceptable types using `Union`:

```python
from typing import Union

def handle(value: Union[int, str]) -> None:
    if isinstance(value, int):
        print(f"Number: {value}")
    else:
        print(f"Text: {value}")

def parse_value(text: str) -> Union[int, float, str]:
    """Parse text into most appropriate type."""
    try:
        return int(text)
    except ValueError:
        try:
            return float(text)
        except ValueError:
            return text
```

### Optional Types

`Optional[T]` is shorthand for `Union[T, None]`, used for nullable types:

```python
from typing import Optional

def find_user(user_id: int) -> Optional[str]:
    """Return username if found, None otherwise."""
    if user_id in database:
        return database[user_id]
    return None

def process(value: Optional[str] = None) -> str:
    """Process value if provided, use default otherwise."""
    if value is None:
        return "default"
    return value.upper()
```

### Modern Python 3.10+ Syntax

Python 3.10+ supports the `|` operator for unions:

```python
def handle(value: int | str) -> None:
    pass

def find_user(user_id: int) -> str | None:
    pass
```

## Callable Types

Annotate function parameters and callbacks:

```python
from typing import Callable

def execute(func: Callable[[int, str], bool]) -> None:
    """Execute a function that takes (int, str) and returns bool."""
    result = func(42, "test")
    print(f"Result: {result}")

def generic_callback(fn: Callable[..., str]) -> None:
    """Execute a function with any arguments that returns str."""
    result = fn()
    print(result)

# Example usage
def validator(code: int, message: str) -> bool:
    return code == 200 and len(message) > 0

execute(validator)
```

Format: `Callable[[arg_types], return_type]`. Use ellipsis (`...`) when argument signatures shouldn't be restricted.

## Any Type

`Any` represents complete flexibility when types cannot be determined:

```python
from typing import Any

def flexible(x: Any) -> Any:
    """Accept and return any type."""
    return x

def process_json(data: dict[str, Any]) -> None:
    """Process JSON with unknown structure."""
    for key, value in data.items():
        print(f"{key}: {value}")
```

**Use `Any` sparingly** - it defeats the purpose of type checking. Prefer more specific types when possible.

## Type Aliases

Create reusable type names for complex types:

```python
from typing import Union
from collections.abc import Sequence

# Simple aliases
UserId = int
Username = str

# Complex aliases
Vector = Sequence[tuple[float, float]]
JSON = dict[str, Any]
IntOrStr = Union[int, str]

# Usage
def get_user(user_id: UserId) -> Username:
    return database[user_id]

def process_vector(v: Vector) -> float:
    return sum(x*x + y*y for x, y in v)

def parse_json(text: str) -> JSON:
    return json.loads(text)
```

## Forward References

Use string literals for self-referential types or forward references:

```python
class Node:
    """Binary tree node."""

    def __init__(self, value: int, left: 'Node | None' = None, right: 'Node | None' = None):
        self.value = value
        self.left = left
        self.right = right

    def add_child(self, child: 'Node') -> None:
        """Add a child node."""
        if self.left is None:
            self.left = child
        elif self.right is None:
            self.right = child

# Python 3.7+ can use 'from __future__ import annotations' to avoid quotes
from __future__ import annotations

class LinkedList:
    def __init__(self, value: int, next: LinkedList | None = None):
        self.value = value
        self.next = next
```

## Avoiding Runtime Overhead

Use `TYPE_CHECKING` for import-only type hints to avoid runtime overhead:

```python
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    # These imports only happen during type checking, not at runtime
    from expensive_module import ExpensiveClass
    from collections.abc import Sequence

def process(items: 'Sequence[ExpensiveClass]') -> None:
    """Process items without importing ExpensiveClass at runtime."""
    pass
```

## Common Patterns

### Default Arguments

```python
def greet(name: str, greeting: str = "Hello") -> str:
    return f"{greeting}, {name}!"
```

### Variable Annotations

```python
# Declare variable types
count: int = 0
names: list[str] = []
cache: dict[str, Any] = {}

# Without initial value
user_id: int
config: dict[str, str]
```

### Class Attributes

```python
class Config:
    """Application configuration."""

    # Class variable with type annotation
    max_connections: int = 100
    timeout: float = 30.0
    debug: bool = False

    def __init__(self, host: str, port: int):
        # Instance variables
        self.host: str = host
        self.port: int = port
```

### Generator Types

```python
from collections.abc import Iterator, Generator

def count_up(n: int) -> Iterator[int]:
    """Simple iterator."""
    for i in range(n):
        yield i

def process_items() -> Generator[str, None, int]:
    """Generator with yield, send, and return types."""
    count = 0
    for item in data:
        yield item
        count += 1
    return count
```

### Overload for Multiple Signatures

```python
from typing import overload

@overload
def process(value: int) -> str: ...

@overload
def process(value: str) -> int: ...

def process(value: int | str) -> int | str:
    """Process int or str with different return types."""
    if isinstance(value, int):
        return str(value)
    return len(value)
```

## Best Practices

1. **Start with public APIs**: Annotate public functions and classes first
2. **Use abstract types for parameters**: `Sequence` instead of `list`, `Mapping` instead of `dict`
3. **Be specific about return types**: Avoid `Any` when possible
4. **Use type aliases for complex types**: Make code more readable
5. **Prefer `None` over `Optional`**: Use `str | None` instead of `Optional[str]` in modern Python
6. **Don't over-annotate**: Internal implementation details don't always need types
7. **Use type checkers**: Run mypy, pyright, or pyre to catch type errors
8. **Gradual typing**: Add types incrementally, focus on high-value areas first

## Key Insight

**Python will remain a dynamically typed language**—type hints enable better tooling, documentation, and IDE support without enforcing runtime constraints. Type checking remains entirely optional and occurs through separate static analysis tools.

Type hints are for developers and tools, not for the Python interpreter itself. They complement, rather than replace, Python's dynamic nature.
