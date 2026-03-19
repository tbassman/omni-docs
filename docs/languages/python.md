# Python

Personal lookup for things I always forget. Terse by design — just enough to jog memory.

---

## pytest

**Always run pytest via `python -m pytest`, not bare `pytest`**
```bash
# Potentially wrong — uses system pytest, which can't find your project's modules:
pytest tests/test_foo.py -v   # ModuleNotFoundError

# Probably more right — uses the venv's pytest and adds the project root to sys.path:
python -m pytest tests/test_foo.py -v
```
Same reason as `python -m` for scripts: the current directory gets added to the module
search path, so imports like `from engine.foo import bar` resolve correctly.

**Share an object across all tests in a file (instantiate once)**
```python
@pytest.fixture(scope="module")
def scenario():
    return MyScenario(config)
```
`scope="module"` = created once per file. Default (no scope) = recreated for every test function.

**Parametrize — one test case per item in a list**
```python
@pytest.mark.parametrize("x,y", [(1, 2), (3, 4)], ids=["case_a", "case_b"])
def test_something(x, y):
    ...
```
pytest runs `test_something` once per tuple — so this produces two test cases:
`test_something[case_a]` with `x=1, y=2` and `test_something[case_b]` with `x=3, y=4`.
Each appears as a separate pass/fail line in `pytest -v`.
IDs control the display name; omit them and pytest auto-generates names like `test_something[1-2]`.

**Fixtures are just functions passed as test parameters**
```python
@pytest.fixture
def my_data():
    return {"key": "value"}   # returned value is injected into the test

def test_foo(my_data):         # pytest sees the name and injects it
    assert my_data["key"] == "value"
```

---

## Testing concepts

**Integration test vs regression test**

These are about *why* you're testing, not *how*:

- **Integration test**: verifies that multiple components work correctly *together* — the emphasis is on the connections between parts. E.g. "does the validation step feed into the calculation step, which correctly produces a CSV."
- **Regression test**: verifies that something that *used to work still works* after a change — the emphasis is on catching accidental breakage. You establish known-correct values, and the test alerts you if a future change causes them to drift.

The same test can be both. A test that runs the full scenario pipeline (integration) and compares output against expected values (regression) is serving both purposes — but "regression test" is the more useful description for what it does day-to-day.

---

## Iteration

**First item matching a condition (with default if none)**
```python
result = next((x for x in items if x > 0), None)
```
The second argument to `next()` is the default — without it, raises `StopIteration`.

---

## Classes / OOP

**Abstract base class — force subclasses to implement a method**
```python
from abc import ABC, abstractmethod

class Base(ABC):
    @abstractmethod
    def calculate(self):
        ...
```
Instantiating a subclass that doesn't implement all `@abstractmethod` methods raises `TypeError`.

---

## Type hints

**List of dicts**
```python
from typing import Any, Dict, List
def foo(items: List[Dict[str, Any]]) -> None:
```

---

## Strings / formatting

**f-string with number formatting**
```python
f"{value:,.2f}"    # 1,234,567.89  (comma thousands, 2 decimal places)
f"{ratio:.1%}"     # 12.3%         (percentage, 1 decimal)
f"{ratio:.2%}"     # 12.34%
```

---

## Files / paths

**Path relative to the current file (not the working directory)**
```python
from pathlib import Path
HERE = Path(__file__).parent
config = HERE / "configs" / "my_config.json"
```
`__file__` is always the path to the Python file being executed — it doesn't change.
The "working directory" is wherever your terminal is when you run the script — it does change.

Example of why this matters:
```
backend/
  tests/
    test_foo.py          ← wants to open tests/fixtures/data.json
    fixtures/
      data.json
```
If you run `pytest` from `backend/`, the working directory is `backend/`.
`open("fixtures/data.json")` fails — there's no `fixtures/` directly inside `backend/`.
`open("tests/fixtures/data.json")` works today but breaks if you ever run from elsewhere.
`Path(__file__).parent / "fixtures" / "data.json"` always works because it's anchored
to where `test_foo.py` lives, not where you happen to be standing in the terminal.

**Running a script that imports from your own project**
```bash
# Wrong — Python doesn't know where to find your project's modules:
python tests/generate_fixture.py   # ModuleNotFoundError

# Right — run as a module from the project root:
python -m tests.generate_fixture   # works
```
`python -m` adds the current directory to the module search path. Always run from the
project root (the directory that contains your top-level packages).

**Read a JSON file**
```python
import json
with open(path) as f:
    data = json.load(f)
```
