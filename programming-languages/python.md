# Python - Language Guide

Python is a versatile, high-level programming language known for readability, rich ecosystem, and broad applicability across scripting, web, data, ML, DevOps, and automation.

---

## Quick Syntax Tour

```python
# Variables and types
count: int = 0
price: float = 9.99
name: str = "Alice"
active: bool = True
items: list[int] = [1, 2, 3]
user: dict[str, str] = {"id": "u1", "name": "Alice"}

# Functions
from typing import Iterable

def total(xs: Iterable[int]) -> int:
    return sum(xs)

# Classes and dataclasses
from dataclasses import dataclass

@dataclass(slots=True)
class User:
    id: int
    name: str

# Exceptions
try:
    1 / 0
except ZeroDivisionError as e:
    print("oops", e)
finally:
    print("done")

# Comprehensions and generators
squares = [x * x for x in range(10)]
unique = {x % 3 for x in range(10)}
keyed = {x: x * x for x in range(5)}

def fibonacci(n: int):
    a, b = 0, 1
    for _ in range(n):
        yield a
        a, b = b, a + b
```

---

## Tooling

- Package managers: `pip`, `pipx`, `poetry`, `pipenv`
- Environments: `venv`, `virtualenv`, `conda`
- Formatters/Linters: `black`, `ruff`, `flake8`, `isort`
- Typing: `mypy`, `pyright`
- Testing: `pytest`, `unittest`, `hypothesis`

### Recommended baseline setup
```bash
python -m venv .venv
. .venv/Scripts/activate  # Windows PowerShell: .venv\Scripts\Activate.ps1
pip install --upgrade pip
pip install black ruff isort mypy pytest
```

pyproject.toml
```toml
[tool.black]
line-length = 100
target-version = ["py311"]

[tool.isort]
profile = "black"

[tool.ruff]
line-length = 100
select = ["E", "F", "I", "UP", "B", "SIM"]

[tool.mypy]
python_version = "3.11"
strict = true
warn_unused_ignores = true
warn_redundant_casts = true
```

---

## Modern Python Patterns

- Use dataclasses or `pydantic` for data models
- Prefer `pathlib` over `os.path`
- Use `with` context managers for resources
- Prefer `subprocess.run([...], check=True)` over `os.system`
- Leverage `typing` (`TypedDict`, `Protocol`, `Literal`, `Self`)

```python
from pathlib import Path
from typing import Protocol

class Repository(Protocol):
    def save(self, data: bytes) -> None: ...

class FileRepository:
    def __init__(self, root: Path) -> None:
        self.root = root

    def save(self, data: bytes) -> None:
        self.root.mkdir(parents=True, exist_ok=True)
        (self.root / "out.bin").write_bytes(data)
```

---

## Async Primer

```python
import asyncio
import httpx

async def fetch(url: str) -> str:
    async with httpx.AsyncClient(timeout=10) as client:
        r = await client.get(url)
        r.raise_for_status()
        return r.text

async def main():
    html = await fetch("https://example.com")
    print(len(html))

if __name__ == "__main__":
    asyncio.run(main())
```

---

## Packaging (PEP 517/518)

```toml
# pyproject.toml
[build-system]
requires = ["setuptools", "wheel"]
build-backend = "setuptools.build_meta"

[project]
name = "my-package"
version = "0.1.0"
description = "Sample package"
readme = "README.md"
requires-python = ">=3.11"
license = {text = "MIT"}
authors = [{ name = "You" }]
dependencies = ["requests>=2.31.0"]
```

Build and install
```bash
pip install build twine
python -m build
pip install dist/my_package-0.1.0-py3-none-any.whl
```

---

## Testing

```python
# tests/test_math.py
import pytest

def add(a: int, b: int) -> int:
    return a + b

def test_add():
    assert add(2, 3) == 5
```

Run:
```bash
pytest -q
```

---

## Best Practices

- Pin dependencies and use lock files (poetry/pip-tools)
- Enforce formatting and linting in CI
- Use type checking in CI
- Keep functions small and pure; handle errors explicitly
- Avoid mutable default arguments
- Structure projects with `src/` layout for packages

