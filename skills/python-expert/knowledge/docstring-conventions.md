# PEP 257: Docstring Conventions

## What is a Docstring?

A docstring is the first string literal in a module, function, class, or method. It becomes the `__doc__` attribute. **Always use `"""triple double quotes"""`** around docstrings.

## One-Line Docstrings

For simple cases, keep docstrings on a single line:

```python
def kos_root():
    """Return the pathname of the KOS root directory."""
    return os.path.join(os.environ['HOME'], '.kos')
```

### Key Guidelines for One-Liners:
- Use triple quotes even for single lines (easier to expand later)
- Closing quotes stay on the same line as opening quotes
- No blank lines before or after
- **Phrase ends with a period**
- **Written as a command**: "Return that" not "Returns the"
- Don't repeat function parameters (introspection provides this)
- Include return value information when applicable

**Examples:**
```python
# Good
def get_user_name():
    """Return the current user's name."""
    return os.getenv('USER')

def is_valid(value):
    """Check if the value is valid."""
    return value > 0

# Bad - wrong style
def get_user_name():
    """Returns the current user's name"""  # Missing period, wrong tense
    return os.getenv('USER')

def add(a, b):
    """add(a, b) -> sum of a and b"""  # Don't repeat signature
    return a + b
```

## Multi-Line Docstrings

Structure: **summary line + blank line + detailed description**

```python
def complex(real=0.0, imag=0.0):
    """Form a complex number.

    Keyword arguments:
    real -- the real part (default 0.0)
    imag -- the imaginary part (default 0.0)
    """
    if imag == 0.0 and real == 0.0:
        return complex_zero
    return Complex(real, imag)
```

### Requirements for Multi-Line Docstrings:
- Summary fits on one line, separated by blank line from rest
- Closing quotes on their own line
- Insert blank line after class docstrings before methods
- Document arguments, return values, side effects, exceptions
- For classes: list public methods and instance variables

**Examples:**
```python
def fetch_data(url, timeout=30, retries=3):
    """Fetch data from a remote URL.

    This function retrieves data from the specified URL with automatic
    retry logic for transient failures. Connection pooling is used for
    efficiency.

    Args:
        url: The URL to fetch data from. Must be a valid HTTP/HTTPS URL.
        timeout: Maximum time in seconds to wait for response (default: 30).
        retries: Number of retry attempts for failed requests (default: 3).

    Returns:
        The response body as a string. Returns None if all retries are
        exhausted.

    Raises:
        ValueError: If the URL is malformed.
        ConnectionError: If the connection cannot be established after
            all retry attempts.
        TimeoutError: If the request times out.

    Example:
        >>> data = fetch_data('https://api.example.com/data')
        >>> print(len(data))
        1024
    """
    pass


class DataProcessor:
    """Process and transform data from various sources.

    This class provides methods for loading, transforming, and saving data.
    It supports multiple input formats including CSV, JSON, and XML.

    Attributes:
        source_type: The type of data source ('csv', 'json', or 'xml').
        cache_enabled: Whether to cache processed results.

    Example:
        >>> processor = DataProcessor('csv')
        >>> processor.load('data.csv')
        >>> processor.transform()
        >>> processor.save('output.json')
    """

    def __init__(self, source_type):
        """Initialize the processor with a specific source type.

        Args:
            source_type: The type of data source to process.
        """
        self.source_type = source_type
        self.cache_enabled = True

    def load(self, filename):
        """Load data from the specified file.

        Args:
            filename: Path to the file to load.

        Raises:
            FileNotFoundError: If the specified file doesn't exist.
            ValueError: If the file format doesn't match source_type.
        """
        pass
```

## Module Docstrings

Module docstrings should describe the module's purpose and list exported classes, exceptions, and functions.

```python
"""Utilities for processing customer data.

This module provides classes and functions for loading, validating,
and transforming customer data from various sources. It includes
support for CSV, JSON, and database sources.

Classes:
    CustomerDataLoader: Loads customer data from files or databases.
    DataValidator: Validates customer data against business rules.
    DataTransformer: Transforms data between different formats.

Functions:
    load_customer_csv: Quick function to load customer data from CSV.
    validate_email: Validate an email address.

Exceptions:
    ValidationError: Raised when data validation fails.
    LoadError: Raised when data loading fails.

Example:
    >>> loader = CustomerDataLoader('customers.csv')
    >>> data = loader.load()
    >>> validator = DataValidator()
    >>> validator.validate(data)
"""

import os
import csv
from typing import List, Dict


class ValidationError(Exception):
    """Raised when customer data fails validation checks."""
    pass
```

## Class Docstrings

Class docstrings should:
- Describe the class purpose
- List public methods and attributes
- Provide usage examples
- Be followed by a blank line before the first method

```python
class Connection:
    """Manage a connection to a remote database.

    This class handles connection pooling, automatic reconnection,
    and query execution with proper error handling.

    Attributes:
        host: The database host address.
        port: The database port number.
        database: The name of the database to connect to.
        is_connected: True if currently connected, False otherwise.

    Methods:
        connect: Establish a connection to the database.
        disconnect: Close the database connection.
        execute: Execute a SQL query and return results.
        commit: Commit the current transaction.
        rollback: Roll back the current transaction.

    Example:
        >>> conn = Connection('localhost', 5432, 'mydb')
        >>> conn.connect()
        >>> results = conn.execute('SELECT * FROM users')
        >>> conn.disconnect()
    """

    def __init__(self, host, port, database):
        """Initialize a new database connection.

        Args:
            host: The database server hostname or IP address.
            port: The port number the database server is listening on.
            database: The name of the database to connect to.
        """
        self.host = host
        self.port = port
        self.database = database
        self.is_connected = False
```

## Package Docstrings

Package docstrings appear in the `__init__.py` file and describe the package's purpose and contents.

```python
"""Data processing utilities package.

This package provides tools for processing, validating, and transforming
customer data. It supports multiple data sources and formats.

Modules:
    loaders: Data loading utilities for various formats.
    validators: Data validation against business rules.
    transformers: Data transformation and format conversion.
    exporters: Export data to various output formats.

Usage:
    from data_utils import loaders, validators

    loader = loaders.CSVLoader('data.csv')
    data = loader.load()

    validator = validators.DataValidator()
    if validator.validate(data):
        print("Data is valid")
"""

from .loaders import CSVLoader, JSONLoader
from .validators import DataValidator
from .transformers import DataTransformer

__all__ = ['CSVLoader', 'JSONLoader', 'DataValidator', 'DataTransformer']
```

## Docstring Indentation

Processing tools strip uniform indentation from all lines after the first, maintaining relative indentation. The indentation of the first line (after the opening quotes) determines the base indentation level.

**Example:**
```python
def example():
    """Summary line.

    This is indented content that will have its
    base indentation stripped, but relative
    indentation preserved:

        - This item is indented
        - So is this one
            - And this is doubly indented

    Back to base level.
    """
    pass
```

## Script Docstrings

Scripts (files meant to be run from command line) should have a docstring explaining usage:

```python
#!/usr/bin/env python3
"""Process customer data files and generate reports.

Usage:
    python process_data.py <input_file> [options]

Options:
    -o, --output FILE    Write output to FILE (default: stdout)
    -f, --format FORMAT  Output format: json, csv, xml (default: json)
    -v, --verbose        Enable verbose output
    -h, --help          Show this help message

Examples:
    python process_data.py customers.csv -o report.json
    python process_data.py data.xml -f csv --verbose
"""

import sys
import argparse


def main():
    """Main entry point for the script."""
    parser = argparse.ArgumentParser(description=__doc__)
    # ... argument parsing
```

## Key Principles

1. **Always use triple double quotes**: `"""docstring"""`
2. **One-liners for simple cases**, multi-line for complex functions
3. **Command voice**: "Return" not "Returns"
4. **Summary + blank line + details** for multi-line docstrings
5. **Document all public APIs**: modules, classes, functions, methods
6. **Include examples** when helpful for understanding usage
7. **Closing quotes on own line** for multi-line docstrings
