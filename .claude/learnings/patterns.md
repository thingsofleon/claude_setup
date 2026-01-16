# Project Patterns

> Auto-updated by SelfReflector agent. Review periodically for accuracy.
> Last update: 2026-01-12

## Architecture Patterns

### Configuration Module Pattern

**Seen in:** ISSUE-env-config

Use a dedicated `config.py` module for environment-based configuration:

1. Load `.env` file at startup using `python-dotenv`
2. Define environment variable names as constants at module level
3. Provide getter functions that:
   - Return parsed values (lists, paths, etc.)
   - Validate paths exist before returning
   - Log warnings for invalid/missing paths
   - Return empty/None gracefully for unconfigured values

```python
# Example structure
ENV_SOME_CONFIG = "SOME_CONFIG"

def load_config(env_path: Optional[Path] = None) -> None:
    load_dotenv(dotenv_path=env_path) if env_path else load_dotenv()

def get_some_config() -> list[str]:
    value = os.getenv(ENV_SOME_CONFIG, "")
    # Parse, validate, return
```

## Code Patterns

### PyQt5 Deferred Loading Pattern

**Seen in:** ISSUE-env-config

When loading resources at PyQt5 app startup, use `QTimer.singleShot()` to defer loading after the window is shown:

```python
def main():
    app = QApplication(sys.argv)
    window = MainWindow()
    window.show()

    # Defer loading to allow window to render first
    QTimer.singleShot(100, lambda: window.load_data())

    # Chain dependent operations with longer delays
    QTimer.singleShot(1000, lambda: window.load_dependent_data())

    sys.exit(app.exec())
```

## Testing Patterns

### Configuration Testing with Fixtures

**Seen in:** ISSUE-env-config

Use `tmp_path` fixture and environment variable mocking for config tests:

```python
@pytest.fixture
def temp_env_file(tmp_path):
    env_file = tmp_path / ".env"
    return env_file

def test_config_loading(temp_env_file, monkeypatch):
    # Write test .env content
    temp_env_file.write_text("SOME_VAR=value")

    # Clear any existing env vars
    monkeypatch.delenv("SOME_VAR", raising=False)

    # Load and test
    load_config(temp_env_file)
    assert get_some_value() == expected
```

## Documentation Patterns

### Google-Style Docstrings

**Seen in:** ISSUE-env-config

Use Google-style docstrings for all public functions:

```python
def get_default_algorithm(algorithm_type: str) -> Optional[str]:
    """Get the default algorithm path for a specific algorithm type.

    Args:
        algorithm_type: The type of algorithm. Valid values are
            "discrimination", "yield_estimation", or "o2d2".

    Returns:
        The absolute path to the algorithm directory if configured and
        exists, None otherwise.

    Example:
        >>> path = get_default_algorithm("discrimination")
        >>> if path:
        ...     load_algorithm(path)
    """
```

---

*This file is automatically updated when workflows complete successfully.*
*Patterns help future workflows maintain consistency.*
