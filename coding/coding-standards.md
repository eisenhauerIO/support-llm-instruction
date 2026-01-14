# Coding Standards

LLM instructions for maintaining code quality and consistency.

## Strategic

### Architecture Principles

- **Configuration-Driven Design**: Control behavior through configuration files, not hardcoded values
- **Modular Architecture**: Keep modules loosely coupled with clear boundaries and single responsibilities
- **Extensibility**: Design for extension without modification of core code

### Configuration Management

Centralize configuration processing in a dedicated module that handles defaults, merging, and validation.

#### Structure

- **Single Entry Point**: Provide one `process_config(path)` function as the public API
- **Defaults File**: Store default values in a separate YAML file (e.g., `config_defaults.yaml`)
- **Deep Merge**: Merge user configuration over defaults recursively, preserving nested structures
- **Complete Configuration Contract**: Downstream code receives a fully validated, merged configuration—it should never handle defaults, merging, or validation

```python
def deep_merge(base: Dict, override: Dict) -> Dict:
    """Deep merge with override values taking precedence."""
    result = copy.deepcopy(base)
    for key, value in override.items():
        if key in result and isinstance(result[key], dict) and isinstance(value, dict):
            result[key] = deep_merge(result[key], value)
        else:
            result[key] = copy.deepcopy(value)
    return result
```

#### Validation

- **Fail Early**: Validate configuration immediately after loading, before any processing
- **Descriptive Errors**: Include expected values and context in validation messages

```python
raise ValueError(
    f"Unexpected parameters for {backend}.{section}.{function_name}: "
    f"{sorted(extra_params)}. Expected: {sorted(expected_params)}"
)
```

#### Schema Extraction

- **Derive from Defaults**: Extract parameter schemas from the defaults file rather than duplicating definitions

#### Extensibility

- **Skip Validation for Custom Functions**: When users register custom functions, bypass strict parameter validation

### Documentation Philosophy

- Document **why** something exists, not merely what it does
- Capture design decisions and trade-offs
- Make constraints and assumptions explicit
- Avoid line-by-line code walkthroughs or obvious behavior descriptions

## Tactical

### Formatting & Style

- **Line Length**: Use line length of 120 to the fullest extent
- **Formatter**: Black with target Python 3.8+
- **Linter**: Ruff with rules E, W, F, I enabled
- **Remove Dead Code**: Remove all dead code unless there is a docstring indicating to keep it

### Import Guidelines

- **Import Ordering**: Standard library first, then third-party, then local imports
- **Lazy Imports**: Use lazy imports for optional dependencies that may not be installed

### Type Hints

- Use type hints for function signatures
- Import types from `typing` module for complex types

```python
from typing import Callable, Dict, List, Optional

def process(data: Dict, callback: Optional[Callable] = None) -> List:
    """Process data with optional callback."""
```

### Docstrings

- Use triple-quoted docstrings for all public functions and classes
- Include Args, Returns, and Raises sections where applicable
- Keep docstrings concise and focused on purpose

```python
def get(self, name: str) -> Callable:
    """
    Get a registered function by name.
    Args:
        name: Name of the function to retrieve
    Returns:
        The registered function
    Raises:
        KeyError: If function is not registered
    """
```

### Error Handling

- Raise descriptive exceptions with helpful error messages
- Include available options when raising KeyError for missing items

```python
if name not in self._registry:
    raise KeyError(
        f"Function '{name}' not registered. "
        f"Available: {list(self._registry.keys())}"
    )
```

### Function Design

- Use explicit parameter names over positional arguments
- Keep functions focused on a single responsibility
- Accept `**kwargs` for extensibility where appropriate

### Naming Conventions

- Use snake_case for functions and variables
- Use PascalCase for classes
- Use UPPER_CASE for constants and configuration keys
- Prefix private methods and attributes with underscore

### Testing

- Use pytest fixtures for common setup
- Tests should be runnable independently
- Group related tests in the same module

### Code Organization

- One class per file when the class is substantial
- Group related functions in the same module
- Use `__init__.py` to expose public API
- Keep implementation details private (prefix with underscore)
