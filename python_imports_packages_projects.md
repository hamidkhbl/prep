# 🐍 Python Imports, `__init__.py` & Larger Project Structure
> Modules, Packages, Namespaces, and Everything That Wires a Python Project Together

---

## TABLE OF CONTENTS

1. [Modules — The Basics](#1-modules--the-basics)
2. [Import Mechanics — How Python Finds Code](#2-import-mechanics--how-python-finds-code)
3. [Import Syntax — All Forms](#3-import-syntax--all-forms)
4. [Packages & `__init__.py`](#4-packages--initpy)
5. [`__all__` — Controlling Public API](#5-__all__--controlling-public-api)
6. [Relative Imports](#6-relative-imports)
7. [Namespace Packages (No `__init__.py`)](#7-namespace-packages-no-initpy)
8. [`__name__` and `__main__`](#8-__name__-and-__main__)
9. [Import Patterns — Best Practices](#9-import-patterns--best-practices)
10. [Common Import Pitfalls](#10-common-import-pitfalls)
11. [Project Layouts — Real-World Structures](#11-project-layouts--real-world-structures)
12. [`pyproject.toml` & `setup.py`](#12-pyprojecttoml--setuppy)
13. [Virtual Environments & Dependency Management](#13-virtual-environments--dependency-management)
14. [The `src/` Layout Pattern](#14-the-src-layout-pattern)
15. [Lazy Imports & Performance](#15-lazy-imports--performance)
16. [Type Checking & `TYPE_CHECKING`](#16-type-checking--type_checking)
17. [Plugin Systems & Dynamic Imports](#17-plugin-systems--dynamic-imports)
18. [Interview Q&A](#18-interview-qa)

---

## 1. Modules — The Basics

### What Is a Module?

A **module** is any `.py` file. When Python imports it, the file is executed top-to-bottom **once**, and the resulting namespace is cached in `sys.modules`.

```python
# math_utils.py  ← this file IS the module
PI = 3.14159

def circle_area(r):
    return PI * r ** 2

class Circle:
    def __init__(self, r):
        self.r = r
```

```python
# main.py
import math_utils

math_utils.PI               # 3.14159
math_utils.circle_area(5)   # 78.53...
c = math_utils.Circle(5)
```

### Module Attributes

```python
import math_utils

math_utils.__name__     # "math_utils"
math_utils.__file__     # "/path/to/math_utils.py"
math_utils.__doc__      # module-level docstring (if any)
math_utils.__dict__     # everything in the module's namespace
dir(math_utils)         # sorted list of attribute names
```

### sys.modules — The Import Cache

```python
import sys

# After first import, module lives here
import math_utils
sys.modules["math_utils"]   # <module 'math_utils' from '...'>

# Importing again does NOT re-execute the file — returns cached version
import math_utils            # instant, no re-execution

# Force re-import (rarely needed, useful in REPL/testing)
import importlib
importlib.reload(math_utils)
```

---

## 2. Import Mechanics — How Python Finds Code

### sys.path — The Search Path

When you write `import foo`, Python searches directories in `sys.path` in order:

```python
import sys
print(sys.path)

# Typical sys.path order:
# 1. '' or the script's own directory (or '' for interactive)
# 2. PYTHONPATH environment variable entries
# 3. Standard library directories
# 4. site-packages (installed third-party packages)
```

```bash
# Inspect from shell
python -c "import sys; print('\n'.join(sys.path))"

# Add to PYTHONPATH at runtime
export PYTHONPATH="/my/project/src:$PYTHONPATH"
```

### The Import Process (Step by Step)

```
import foo
    │
    ├─ 1. Check sys.modules["foo"]  →  found? return cached module
    │
    ├─ 2. Find the module
    │       Search sys.path for:
    │         foo.py           (source module)
    │         foo/             (package with __init__.py)
    │         foo/__init__.py  (regular package)
    │         foo.so / foo.pyd (C extension)
    │
    ├─ 3. Load & compile
    │       Execute foo.py top to bottom
    │       Cache result in sys.modules["foo"]
    │
    └─ 4. Bind name
            Bind "foo" in the caller's local namespace
```

### Compiled Bytecode — `__pycache__`

```
my_project/
├── utils.py
└── __pycache__/
    └── utils.cpython-312.pyc   ← cached bytecode
```

Python auto-creates `.pyc` files to skip re-parsing on subsequent imports. You can safely add `__pycache__/` to `.gitignore`.

---

## 3. Import Syntax — All Forms

```python
# ── Form 1: import module ───────────────────────────────────
import os
import os.path

os.getcwd()
os.path.join("a", "b")    # must use full dotted path

# ── Form 2: from module import name ────────────────────────
from os import getcwd, listdir
from os.path import join, exists

getcwd()      # no prefix needed
join("a","b")

# ── Form 3: import module as alias ──────────────────────────
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt

np.array([1,2,3])    # conventional alias

# ── Form 4: from module import name as alias ─────────────────
from collections import defaultdict as dd
from typing import Optional as Opt

dd(list)

# ── Form 5: from package import submodule ───────────────────
from email import mime
from email.mime import text

# ── Form 6: from module import * (avoid in production) ───────
from math import *   # pulls everything in __all__ (or all public names)
sqrt(4)              # works but pollutes namespace — hard to trace origin

# ── Form 7: Conditional / guard imports ─────────────────────
try:
    import ujson as json    # fast JSON library
except ImportError:
    import json             # fall back to stdlib

# ── Form 8: Import inside a function (lazy / deferred) ───────
def process_image(path):
    from PIL import Image   # only imported when function is called
    return Image.open(path)
```

---

## 4. Packages & `__init__.py`

### What Is a Package?

A **package** is a directory containing an `__init__.py` file. The `__init__.py` is executed when the package is imported — it can be empty, or it can set up the package's public API.

```
mypackage/
├── __init__.py      ← executed on "import mypackage"
├── models.py
├── utils.py
└── services/
    ├── __init__.py
    ├── auth.py
    └── email.py
```

### Empty `__init__.py`

```python
# __init__.py  (empty)
# The directory is a package; all submodules must be imported explicitly.

# Usage:
import mypackage.models
from mypackage import utils
from mypackage.services import auth
```

### `__init__.py` as Public API Gateway

```python
# mypackage/__init__.py

# Re-export the things users should access at the top level
from .models  import User, Product
from .utils   import format_date, slugify
from .services.auth import authenticate

# Version
__version__ = "1.2.0"
__author__  = "Hamid"

# Usage after this:
from mypackage import User, authenticate   # clean top-level access
import mypackage
mypackage.__version__   # "1.2.0"
```

### Nested Package `__init__.py`

```python
# mypackage/services/__init__.py

from .auth  import authenticate, logout
from .email import send_email

# Now callers can do:
from mypackage.services import authenticate
# instead of:
from mypackage.services.auth import authenticate
```

### What Goes in `__init__.py`?

| Do ✅ | Avoid ❌ |
|---|---|
| Re-export public API with `from .x import Y` | Heavy logic or slow code (slows every import) |
| Set `__version__`, `__author__` | Circular imports |
| Lazily import expensive submodules | `import *` without `__all__` |
| Register plugins or run lightweight config | Side effects that change global state |

---

## 5. `__all__` — Controlling Public API

```python
# utils.py

__all__ = ["format_date", "slugify"]  # only these are exported

def format_date(dt):   ...   # public ✅
def slugify(text):     ...   # public ✅
def _internal_helper(): ...  # private by convention
def also_private():    ...   # also private — not in __all__
```

```python
from utils import *          # imports ONLY format_date, slugify
from utils import _internal  # still works — __all__ only affects *
```

### `__all__` in `__init__.py`

```python
# mypackage/__init__.py
from .models  import User, Product
from .utils   import format_date

__all__ = ["User", "Product", "format_date"]
# Documents the intended public surface of the package
```

### Best Practice

Always define `__all__` in modules intended to be used as libraries. It serves as living documentation of your public API and prevents accidental exposure of private helpers.

---

## 6. Relative Imports

Relative imports use `.` notation and only work **inside packages** (not scripts).

```
mypackage/
├── __init__.py
├── models.py
├── utils.py
└── services/
    ├── __init__.py
    ├── auth.py
    └── email.py
```

```python
# mypackage/services/auth.py

# Absolute import (works anywhere)
from mypackage.models import User
from mypackage.utils  import format_date

# Relative import (. = current package = mypackage/services/)
from .   import email               # sibling module
from ..  import utils               # parent package (mypackage/)
from ..models import User           # from parent's models.py
from ..utils  import format_date    # from parent's utils.py

# Two levels up
from ... import something           # grandparent package
```

### Relative vs Absolute — When to Use Each

```python
# ✅ Relative — inside a package, referring to siblings or parents
# Benefit: refactoring-friendly (renaming the package doesn't break internals)
from .utils import helper

# ✅ Absolute — always works, more explicit, preferred for deep cross-package refs
from mypackage.utils import helper

# ❌ Never use relative imports in scripts (files run directly)
# Running auth.py directly: "from .. import utils" → ImportError
```

---

## 7. Namespace Packages (No `__init__.py`)

Python 3.3+ supports **namespace packages** — directories without `__init__.py`. Useful for splitting a logical package across multiple directories or repositories.

```
# Split across two locations on sys.path:
/path/a/myns/foo.py
/path/b/myns/bar.py

# Both accessible as:
from myns import foo
from myns import bar
```

```python
import myns.foo   # works even though no myns/__init__.py exists
import myns.bar
```

**Use namespace packages when:**
- A large organization splits a package across repos (`company.team1`, `company.team2`)
- You want zero-overhead empty `__init__.py` files
- You're building a plugin system where plugins are dropped into a directory

**Use regular packages (with `__init__.py`) when:**
- You need to define a public API
- You need `__version__` or `__all__`
- You want `from mypackage import SomeClass` to work

---

## 8. `__name__` and `__main__`

Every module has a `__name__` attribute:
- When **imported**: `__name__` = the module's dotted name (e.g. `"mypackage.utils"`)
- When **run directly**: `__name__` = `"__main__"`

```python
# mypackage/utils.py

def main():
    print("Running utils directly")

if __name__ == "__main__":
    main()
# Only runs when: python mypackage/utils.py
# Does NOT run when: import mypackage.utils
```

### `__main__.py` — Runnable Packages

```
mypackage/
├── __init__.py
├── __main__.py    ← entry point when package is run directly
├── cli.py
└── ...
```

```python
# mypackage/__main__.py
from .cli import run

if __name__ == "__main__":
    run()
```

```bash
# Run the package directly:
python -m mypackage        # executes __main__.py
python -m mypackage --help # passes args to cli
```

This is how tools like `python -m pytest`, `python -m http.server`, and `python -m pip` work.

---

## 9. Import Patterns — Best Practices

### Canonical Import Order (PEP 8)

```python
# 1. Standard library
import os
import sys
from pathlib import Path
from typing import Optional

# 2. Third-party libraries
import requests
import pytest
from pydantic import BaseModel

# 3. Local application / package imports
from mypackage import User
from mypackage.services import auth
from .utils import helper

# Separate each group with a blank line
# Use isort to autoformat: pip install isort
```

### Avoid Circular Imports — Design Patterns

```
# ❌ Circular import problem:
# a.py: from b import B
# b.py: from a import A   ← imports a while a is still loading
```

**Solution 1: Move shared code to a third module**
```python
# common.py — no imports from a or b
class Base: ...

# a.py
from common import Base
class A(Base): ...

# b.py
from common import Base
class B(Base): ...
```

**Solution 2: Import inside a function**
```python
# a.py
def get_b():
    from b import B    # deferred — a is fully loaded by the time this runs
    return B()
```

**Solution 3: `TYPE_CHECKING` guard (type hints only)**
```python
from __future__ import annotations   # makes all annotations strings (lazy)
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from b import B   # only evaluated by type checkers, not at runtime

def process(item: "B") -> None: ...
```

### Import Aliasing Conventions

```python
import numpy           as np
import pandas          as pd
import matplotlib.pyplot as plt
import seaborn         as sns
import tensorflow      as tf
import sqlalchemy      as sa
import pytest                      # no alias needed — short enough
from   datetime import datetime    # bring class directly
from   pathlib  import Path        # bring class directly
from   typing   import (           # group related typing imports
    Any, Dict, List, Optional,
    Tuple, Union, TYPE_CHECKING
)
```

---

## 10. Common Import Pitfalls

### 1. Shadowing Standard Library or Third-Party Names

```python
# ❌ BAD — naming your file the same as a stdlib module
# json.py in your project shadows the built-in json module!

# Inside json.py:
import json   # imports ITSELF → RecursionError or AttributeError

# Fix: rename your file to something unique
# data_parser.py, config_json.py, etc.
```

### 2. Star Import Clobbering Names

```python
from os.path import *
from shutil import *    # may overwrite names from os.path!

# You never know what you imported — avoid * in production code
```

### 3. Import Side Effects

```python
# ❌ BAD — module runs heavy code on import
# slow_module.py
print("Loading slow module...")     # side effect
download_large_file()               # expensive on every import!
DB_CONNECTION = connect_to_db()     # connection on import

# ✅ GOOD — defer expensive work
def get_connection():
    global _conn
    if _conn is None:
        _conn = connect_to_db()
    return _conn

_conn = None
```

### 4. Modifying `sys.path` Directly (Fragile)

```python
# ❌ Fragile — hardcoded absolute path
import sys
sys.path.insert(0, "/home/hamid/projects/myproject/src")

# ✅ Better — compute path relative to current file
from pathlib import Path
sys.path.insert(0, str(Path(__file__).parent.parent / "src"))

# ✅ Best — install the package properly (editable install)
# pip install -e .
# Then never touch sys.path manually
```

### 5. Circular Import via `__init__.py`

```python
# ❌ Circular via __init__.py
# mypackage/__init__.py
from .services.auth import authenticate   # auth.py imports from models.py
                                          # models.py imports from __init__.py → circular

# ✅ Fix: import from the submodule directly, not from __init__
# In auth.py:
from mypackage.models import User   # absolute, skips __init__
# or
from ..models import User           # relative, skips __init__
```

### 6. Importing Before Package Is On sys.path

```bash
# Running from wrong directory
cd /home/hamid
python myproject/src/mypackage/main.py   # ImportError: no module named mypackage

# Fix: run as module from project root
cd /home/hamid/myproject
python -m src.mypackage.main
# or install the package first:
pip install -e .
python -m mypackage.main
```

---

## 11. Project Layouts — Real-World Structures

### Flat Layout (Small Projects / Scripts)

```
my_tool/
├── my_tool.py          # single-file application
├── requirements.txt
└── README.md
```

### Standard Layout (Medium Projects)

```
my_project/
├── my_project/             # package (same name as project)
│   ├── __init__.py
│   ├── __main__.py         # python -m my_project entry point
│   ├── models/
│   │   ├── __init__.py
│   │   ├── user.py
│   │   └── product.py
│   ├── services/
│   │   ├── __init__.py
│   │   ├── auth.py
│   │   └── email.py
│   ├── utils/
│   │   ├── __init__.py
│   │   ├── date_utils.py
│   │   └── validators.py
│   └── config.py
├── tests/
│   ├── __init__.py         # (optional in modern pytest)
│   ├── conftest.py
│   ├── test_models.py
│   ├── test_services.py
│   └── unit/
│       ├── test_auth.py
│       └── test_email.py
├── docs/
├── pyproject.toml          # or setup.cfg / setup.py
├── requirements.txt
├── requirements-dev.txt
├── .env.example
├── .gitignore
└── README.md
```

### SDET / Automation Framework Layout

```
automation_framework/
├── framework/
│   ├── __init__.py
│   ├── api/
│   │   ├── __init__.py
│   │   ├── client.py       # base API client
│   │   └── endpoints.py    # endpoint definitions
│   ├── ui/
│   │   ├── __init__.py
│   │   ├── base_page.py
│   │   ├── driver_factory.py
│   │   └── pages/
│   │       ├── __init__.py
│   │       ├── login_page.py
│   │       └── dashboard_page.py
│   ├── utils/
│   │   ├── __init__.py
│   │   ├── config.py
│   │   ├── logger.py
│   │   └── data_factory.py
│   └── fixtures/
│       ├── __init__.py
│       └── shared_fixtures.py
├── tests/
│   ├── conftest.py
│   ├── api/
│   │   ├── test_users.py
│   │   └── test_auth.py
│   ├── ui/
│   │   ├── test_login.py
│   │   └── test_checkout.py
│   └── integration/
│       └── test_end_to_end.py
├── data/
│   ├── test_users.json
│   └── schemas/
│       └── user_schema.json
├── reports/
├── pyproject.toml
└── conftest.py             # root conftest (session-wide fixtures)
```

---

## 12. `pyproject.toml` & `setup.py`

### `pyproject.toml` (Modern Standard — PEP 517/518)

```toml
# pyproject.toml

[build-system]
requires      = ["setuptools>=68", "wheel"]
build-backend = "setuptools.backends.legacy:build"

[project]
name        = "my-project"
version     = "1.0.0"
description = "My awesome project"
readme      = "README.md"
license     = {file = "LICENSE"}
authors     = [{name = "Hamid", email = "hamid@example.com"}]
requires-python = ">=3.10"

dependencies = [
    "requests>=2.31",
    "pydantic>=2.0",
]

[project.optional-dependencies]
dev  = ["pytest", "pytest-cov", "black", "mypy", "isort"]
test = ["pytest", "pytest-mock", "responses"]

[project.scripts]
my-tool = "my_project.cli:main"   # creates a CLI command

[tool.pytest.ini_options]
testpaths   = ["tests"]
addopts     = "-v --tb=short --cov=my_project --cov-report=term-missing"
markers     = [
    "smoke: quick sanity checks",
    "slow: tests that take > 5s",
]

[tool.black]
line-length = 100
target-version = ["py310", "py311", "py312"]

[tool.isort]
profile = "black"

[tool.mypy]
python_version = "3.11"
strict         = true
ignore_missing_imports = true

[tool.coverage.run]
source = ["my_project"]
omit   = ["*/tests/*", "*/__main__.py"]
```

### Editable Install (Development Mode)

```bash
# Install package in editable mode — changes to source are reflected immediately
pip install -e .
pip install -e ".[dev]"     # include dev extras

# Now you can do:
from my_project import anything   # works from anywhere in the venv
python -m my_project              # runs __main__.py
```

---

## 13. Virtual Environments & Dependency Management

### venv (Built-in)

```bash
# Create
python -m venv .venv

# Activate
source .venv/bin/activate          # macOS / Linux
.venv\Scripts\activate             # Windows PowerShell
.venv\Scripts\activate.bat         # Windows CMD

# Deactivate
deactivate

# Verify
which python      # should point inside .venv/
python --version
pip list

# Install
pip install requests pytest
pip install -r requirements.txt
pip install -e .                   # editable install

# Freeze
pip freeze > requirements.txt
```

### pip-tools (Reproducible Builds)

```bash
# requirements.in — high-level dependencies (source of truth)
requests>=2.31
pydantic>=2.0

# Compile to pinned requirements.txt
pip-compile requirements.in

# Sync environment to exact pins
pip-sync requirements.txt
```

### uv (Modern, Ultra-Fast — 2024+)

```bash
# Install
pip install uv

# Create venv and install
uv venv
uv pip install -r requirements.txt
uv pip install -e ".[dev]"
uv run pytest                      # run in venv without activating
```

### Poetry (Dependency + Build Tool)

```bash
# Initialize
poetry init
poetry add requests pydantic
poetry add --group dev pytest mypy

# Install
poetry install
poetry install --with dev

# Run
poetry run pytest
poetry run python -m my_project

# Export to requirements.txt
poetry export -f requirements.txt --output requirements.txt
```

### `.gitignore` Essentials

```gitignore
# Virtual environments
.venv/
venv/
env/

# Bytecode
__pycache__/
*.py[cod]
*.pyo

# Build artifacts
dist/
build/
*.egg-info/

# Test & coverage
.pytest_cache/
.coverage
htmlcov/

# Environment
.env
.env.*
!.env.example
```

---

## 14. The `src/` Layout Pattern

The `src/` layout prevents accidentally importing the local (uninstalled) package and forces you to always install it properly. Recommended for libraries meant to be distributed.

```
my_project/                 ← repo root
├── src/
│   └── my_project/         ← package lives under src/
│       ├── __init__.py
│       ├── models.py
│       └── utils.py
├── tests/                  ← tests are OUTSIDE src/
│   ├── conftest.py
│   └── test_models.py
├── pyproject.toml
└── README.md
```

```toml
# pyproject.toml — tell setuptools where to find packages
[tool.setuptools.packages.find]
where = ["src"]
```

```bash
# Must install before tests can import it
pip install -e .

# pytest can now find it
pytest tests/
```

**Why `src/` layout?**

```
Without src/:                   With src/:
python                          python
>>> import my_project           >>> import my_project
# imports LOCAL directory       # imports INSTALLED package
# even if not installed!        # forces editable install
# masks missing __init__.py     # catches packaging bugs early
```

---

## 15. Lazy Imports & Performance

### importlib — Programmatic Import

```python
import importlib

# Import a module by string name
mod = importlib.import_module("json")
mod.dumps({"key": "value"})

# Import from a package
auth = importlib.import_module(".auth", package="mypackage.services")

# Reload a module (useful in tests or REPL)
importlib.reload(mod)
```

### Lazy Import Pattern

```python
# Heavy module — don't import until needed
_numpy = None

def get_numpy():
    global _numpy
    if _numpy is None:
        import numpy
        _numpy = numpy
    return _numpy

def compute(data):
    np = get_numpy()
    return np.array(data).mean()
```

### `__getattr__` at Module Level (3.7+ lazy submodule trick)

```python
# mypackage/__init__.py

def __getattr__(name):
    """Lazily import submodules on first attribute access."""
    if name == "heavy_module":
        from . import heavy_module
        globals()["heavy_module"] = heavy_module
        return heavy_module
    raise AttributeError(f"module {__name__!r} has no attribute {name!r}")

# Usage:
import mypackage
mypackage.heavy_module.do_something()   # imported on first access only
```

---

## 16. Type Checking & `TYPE_CHECKING`

### The Problem

```python
# a.py
from b import B    # runtime import

# b.py
from a import A    # runtime import → circular!
```

### The Solution

```python
# a.py
from __future__ import annotations   # ALL annotations become strings (PEP 563)
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from b import B   # only imported when a type checker (mypy) runs
                      # NOT at runtime → no circular import

def process(item: B) -> None:   # "B" is a string at runtime — no error
    ...
```

### `from __future__ import annotations` Explained

```python
# Without it (Python 3.9 and earlier):
def func(x: MyClass) -> MyClass:   # MyClass must be defined before this line!
    ...

# With it:
from __future__ import annotations
def func(x: MyClass) -> MyClass:   # MyClass is a string "MyClass" at runtime
    ...                              # evaluated lazily → no forward reference problem
```

### Forward References (Without `__future__`)

```python
class Node:
    def __init__(self, next: "Node" = None):  # quote the name → string annotation
        self.next = next

class Tree:
    left:  "Tree"   # string annotation
    right: "Tree"
```

---

## 17. Plugin Systems & Dynamic Imports

### importlib-based Plugin Loader

```python
# plugin_loader.py
import importlib
import pkgutil
from pathlib import Path

class PluginRegistry:
    def __init__(self):
        self._plugins = {}

    def register(self, name: str, plugin):
        self._plugins[name] = plugin

    def get(self, name: str):
        if name not in self._plugins:
            raise KeyError(f"Unknown plugin: {name}")
        return self._plugins[name]

    def load_from_package(self, package_name: str):
        """Auto-discover and load all modules in a package."""
        package = importlib.import_module(package_name)
        package_path = Path(package.__file__).parent

        for finder, module_name, _ in pkgutil.iter_modules([str(package_path)]):
            full_name = f"{package_name}.{module_name}"
            module    = importlib.import_module(full_name)

            # Convention: each plugin module exposes a PLUGIN dict
            if hasattr(module, "PLUGIN"):
                self.register(module_name, module.PLUGIN)

registry = PluginRegistry()
registry.load_from_package("myapp.plugins")
```

### Entry Points (setuptools / importlib.metadata)

```toml
# pyproject.toml — declare plugins as entry points
[project.entry-points."myapp.plugins"]
csv_reporter  = "myapp.plugins.csv:CsvReporter"
html_reporter = "myapp.plugins.html:HtmlReporter"
```

```python
# Discover entry-point plugins at runtime
from importlib.metadata import entry_points

def load_plugins(group: str) -> dict:
    plugins = {}
    for ep in entry_points(group=group):
        plugins[ep.name] = ep.load()    # imports and returns the class/function
    return plugins

reporters = load_plugins("myapp.plugins")
reporters["csv_reporter"]()   # CsvReporter instance
```

---

## 18. Interview Q&A

**Q: What happens when you `import` a module twice?**
> The first import executes the module file and caches it in `sys.modules`. The second import simply returns the cached object — the file is NOT re-executed. This makes imports idempotent and fast. You can force a re-import with `importlib.reload()`, but this is rarely needed outside of REPL sessions.

**Q: What is `__init__.py` and what should go in it?**
> `__init__.py` marks a directory as a Python package and is executed when the package is first imported. It should contain re-exports that define the package's public API (`from .module import Class`), metadata like `__version__`, and optionally lazy imports. It should NOT contain heavy computation, database connections, or anything that causes side effects on import.

**Q: What is the difference between a package and a module?**
> A module is a single `.py` file. A package is a directory of modules with an `__init__.py`. A package can contain sub-packages, creating a hierarchy. When you import a package, Python executes its `__init__.py`. Both are objects in `sys.modules` after import.

**Q: What is `__all__` and when should you use it?**
> `__all__` is a list of strings that defines what names are exported when someone does `from module import *`. It also serves as the documented public API — type checkers and IDEs use it. You should define it in any module intended for external use, listing only what consumers should depend on.

**Q: What are relative imports and when can't you use them?**
> Relative imports use `.` notation (`from . import utils`, `from ..models import User`) to import relative to the current package. They only work inside packages — if you run a `.py` file directly (as a script), Python sets `__name__ = "__main__"` and relative imports fail with `ImportError: attempted relative import with no known parent package`. The fix is to run as a module: `python -m mypackage.submodule`.

**Q: How do you resolve circular imports?**
> Three approaches: (1) refactor shared code into a third module that neither of the circular pair imports from; (2) move the import inside a function so it's deferred until the module is fully loaded; (3) use `TYPE_CHECKING` to guard imports that are only needed for type hints. The root cause is usually poor separation of concerns — circular imports are a design smell.

**Q: What is an editable install and why is it useful?**
> `pip install -e .` installs the package in "editable" mode — instead of copying files to `site-packages`, Python adds the source directory to `sys.path`. Changes to source files are immediately reflected without reinstalling. Essential for development: you can run tests, scripts, and the REPL all using the same source tree.

**Q: What is the `src/` layout and why is it recommended?**
> In the `src/` layout, the package lives under a `src/` directory rather than at the repo root. This prevents Python from accidentally importing the local directory instead of the installed package, which can mask missing `__init__.py` files and packaging bugs. It forces you to do an editable install, ensuring tests always run against the same code that would be distributed.

**Q: What is `__main__.py` used for?**
> Placing a `__main__.py` inside a package lets you run the package directly with `python -m mypackage`. This is how standard library tools like `python -m pytest`, `python -m http.server`, and `python -m pip` work. It's the cleanest way to provide a CLI entry point that also works with `python -m`.

**Q: What's the difference between `sys.path` manipulation and an editable install?**
> Manually inserting into `sys.path` is fragile — it's tied to the machine's directory structure, breaks when you move the project, and must be repeated in every entry point. An editable install (`pip install -e .`) makes the package a real first-class import on the system, works from any working directory, and is reproducible across environments. Always prefer editable installs over `sys.path` hacks.

---

## Quick Reference Cheatsheet

```python
# Import forms
import os                              # module, access via os.xxx
from os import path                    # bring name into scope
from os import path as p               # with alias
from os import *                       # all public names (avoid)
from . import sibling                  # relative: sibling module
from .. import parent_thing            # relative: parent package
from ..models import User              # relative: from parent's submodule

# Module introspection
module.__name__                        # "mypackage.utils"
module.__file__                        # path to .py file
module.__doc__                         # module docstring
module.__all__                         # public API list
sys.modules["module_name"]             # cached module object
importlib.reload(module)               # re-execute module file

# Package signals
__init__.py                            # regular package marker
__main__.py                            # python -m package entry point
__all__ = ["A", "B"]                   # public API declaration
__version__ = "1.0.0"                  # package version

# Run as module
python -m mypackage                    # runs __main__.py
python -m pytest tests/               # runs pytest as module
python -m mypackage.cli --help        # runs specific submodule

# Install
pip install -e .                       # editable install
pip install -e ".[dev,test]"           # with extras
pip install -r requirements.txt        # from requirements file
```

---

*Last updated: April 2026 | Python 3.10+ | PEP 8, PEP 517, PEP 518, PEP 563*
