# Python Expert Skill

Expert guidance on Python best practices, style guides, type hints, and code quality based on authoritative sources.

## What This Skill Provides

Comprehensive Python programming guidance based on:
- **Google's Python Style Guide** - Naming, imports, formatting, documentation
- **PEP 8** - Official Python style guide
- **PEP 257** - Docstring conventions
- **PEP 484** - Type hints and annotations

## Usage

Invoke the skill using `/python-expert` followed by your question:

```bash
/python-expert How should I name my class variables?
/python-expert What's the correct docstring format?
/python-expert Should I use List[int] or list[int]?
/python-expert How do I format long function calls?
```

## Example Questions

### Style & Formatting
- "What's the proper indentation for continuation lines?"
- "How should I order my imports?"
- "Where should I use blank lines in my code?"
- "What's the maximum line length?"

### Naming Conventions
- "How should I name private methods?"
- "What's the difference between _single and __double underscore?"
- "Should constants be uppercase?"
- "How do I name exception classes?"

### Documentation
- "How do I write a one-line docstring?"
- "What sections should a function docstring include?"
- "How do I document class attributes?"
- "What's the format for the Args section?"

### Type Hints
- "When should I use Optional vs Union?"
- "What's the difference between List and Sequence?"
- "How do I annotate a function that returns nothing?"
- "Should I use list[int] or List[int] in Python 3.10+?"

### Best Practices
- "How should I handle exceptions?"
- "When should I use comprehensions vs loops?"
- "How do I check for None values?"
- "Should I use lambda or def for function assignments?"

## Knowledge Base

The skill includes four comprehensive guides:

1. **google-style-guide.md** - Google's Python Style Guide
   - Language rules, imports, exceptions
   - Naming standards and conventions
   - Documentation and type annotations
   - Best practices and philosophy

2. **pep8-style-guide.md** - PEP 8 Official Style Guide
   - Code layout and indentation
   - Whitespace rules
   - Naming conventions
   - Comments and programming recommendations

3. **docstring-conventions.md** - PEP 257 Docstrings
   - One-line vs multi-line docstrings
   - Module, class, and function documentation
   - Proper formatting and structure
   - Examples and best practices

4. **type-hints-guide.md** - PEP 484 Type Hints
   - Basic types and generics
   - TypeVar and Union types
   - Callable annotations
   - Modern Python 3.9+ and 3.10+ syntax

## Making This Skill Global

This skill is currently in your project's `.claude/skills/` directory. To make it available across all projects:

### Option 1: Symlink (Recommended)
```bash
# Create a global skills directory if it doesn't exist
mkdir -p ~/.claude/skills

# Create a symlink
ln -s /Users/annabaker/code/.claude/skills/python-expert ~/.claude/skills/python-expert
```

### Option 2: Copy
```bash
# Create a global skills directory if it doesn't exist
mkdir -p ~/.claude/skills

# Copy the entire skill directory
cp -r /Users/annabaker/code/.claude/skills/python-expert ~/.claude/skills/python-expert
```

## Development

### Structure
```
python-expert/
├── SKILL.md                           # Skill definition and instructions
├── README.md                          # This file
└── knowledge/
    ├── google-style-guide.md         # Google's Python Style Guide
    ├── pep8-style-guide.md           # PEP 8
    ├── docstring-conventions.md      # PEP 257
    └── type-hints-guide.md           # PEP 484
```

### Updating the Knowledge Base

To update the guides with new information:

1. Edit the relevant markdown file in `knowledge/`
2. Add new sections or examples as needed
3. Update the SKILL.md if new topics are added
4. Test the skill by invoking it with questions about the new content

## Tips

- Be specific with your questions for more targeted answers
- Ask about specific code patterns to get concrete examples
- Reference PEPs by name (e.g., "according to PEP 8...")
- Ask for both good and bad examples to understand anti-patterns
- Use this skill during code reviews to verify style compliance

## References

- [Google Python Style Guide](https://google.github.io/styleguide/pyguide.html)
- [PEP 8 – Style Guide for Python Code](https://peps.python.org/pep-0008/)
- [PEP 257 – Docstring Conventions](https://peps.python.org/pep-0257/)
- [PEP 484 – Type Hints](https://peps.python.org/pep-0484/)
