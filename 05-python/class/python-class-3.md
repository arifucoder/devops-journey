# 🐍 Python Class 3 — Module, Unit Testing, CI ও YAML

> এই ক্লাসে আছে: Python Module (import system), Virtual Environment, Unit Testing (pytest ও unittest), GitHub Actions দিয়ে CI/CD, আর YAML ফাইল।

---

## 📦 ১. Python Module System

Python **modular system** এ কাজ করে — অর্থাৎ কোডকে ছোট ছোট আলাদা ফাইলে ভাগ করে রাখা যায়, আর দরকার মতো এক ফাইল থেকে আরেক ফাইলে import করা যায়। প্রতিটা `.py` ফাইল আসলে একটা **module**।

### ধাপ ১: একটা module তৈরি করা → `pymodules.py`

```python
def add(a, b):
    """Returns the sum of two numbers."""
    return a + b

def subtract(a, b):
    """Returns the difference of two numbers."""
    return a - b

def multiply(a, b):
    """Returns the product of two numbers."""
    return a * b

def divide(a, b):
    """Returns the quotient. Handles division by zero."""
    if b == 0:
        return "Error: Cannot divide by zero"
    return a / b

def remainder(a, b):
    """Returns the remainder (modulus)."""
    if b == 0:
        return "Error: Cannot calculate remainder of zero"
    return a % b

name = 'Nure'
```

### ধাপ ২: অন্য ফাইলে module ব্যবহার করা → `app.py`

```python
# import করার কয়েকটা উপায় আছে:

# উপায় ১: পুরো module import (সব function চলে আসে)
# import pymodules
# pymodules.add(2, 3)

# উপায় ২: শুধু দরকারি function import, এবং তাকে নতুন নাম (alias) দেওয়া
from pymodules import add as joge
print(joge(2, 3))
```

### দুইভাবে import করার তুলনা

```python
# --- পুরো module import করলে ---
import pymodules

nodes = 10
processes_per_node = 4
total_capacity = pymodules.multiply(nodes, processes_per_node)

total_tasks = 43
tasks_per_node = pymodules.divide(total_tasks, nodes)
leftover_tasks = pymodules.remainder(total_tasks, nodes)

print("--- Infrastructure Report ---")
print(f"Total Capacity: {total_capacity} slots")
print(f"Tasks per Node: {tasks_per_node}")
print(f"Tasks remaining for manual queue: {leftover_tasks}")
```

> 💡 **মূল কথা:**
> - `import pymodules` করলে ওই ফাইলের **সবকিছু** চলে আসে (ব্যবহারের সময় `pymodules.add()` এভাবে লিখতে হয়)।
> - `from pymodules import add` করলে শুধু **দরকারি জিনিসটাই** আসে (সরাসরি `add()` লেখা যায়)।
> - **Best practice:** যা প্রয়োজন শুধু সেটাই import করা — এতে কোড পরিষ্কার থাকে আর মেমরি কম লাগে।

---

## 🧪 ২. Testing কেন দরকার?

Professional developer রা কোড লেখার পর সেটা ঠিকভাবে কাজ করছে কিনা যাচাই করতে **test** লেখেন। অনেক ধরনের testing আছে, তবে সবচেয়ে বেসিক আর জনপ্রিয় হলো **unit testing** — যেখানে কোডের ছোট ছোট অংশ (যেমন প্রতিটা function) আলাদাভাবে পরীক্ষা করা হয়।

**Industry তে কাজের ধরন:**
1. Developer নিজের computer এ test লেখেন ও run করেন।
2. তারপর কোড GitHub / GitLab (বা অন্য কোথাও) push করেন।
3. Push করার সাথে সাথে test **automatically** চলে (এটাই CI — নিচে আছে)।

---

## 🌐 ৩. Virtual Environment (venv)

বিভিন্ন প্রজেক্টে বিভিন্ন version এর package/library লাগতে পারে। এক প্রজেক্টের package যেন আরেক প্রজেক্টের সাথে সংঘর্ষ (conflict) না করে, তার জন্য আলাদা **virtual environment** বানানো হয় — এটা একটা আলাদা, আলাদা করে রাখা জায়গা।

```bash
# 1. Virtual environment তৈরি করা
# (.venv বা nur বা যেকোনো নাম দেওয়া যায়)
python3 -m venv .venv

# 2. Activate করা (চালু করা)
source .venv/bin/activate

# 3. এখন এখানে দরকারি package install করা যায়
pip3 install pytest

# 4. কাজ শেষে deactivate করা (বন্ধ করা)
deactivate
```

> 💡 activate করার পর যা কিছু install করবে, সব ওই virtual environment এর ভেতরেই থাকবে — তোমার মূল system পরিষ্কার থাকবে।

---

## ✅ ৪. pytest দিয়ে Unit Testing

সবচেয়ে সহজ ও জনপ্রিয় testing tool হলো **pytest**।

### `calculator.py` (যে কোড টেস্ট করব)

```python
def add(a, b):
    return a + b

def subtract(a, b):
    return a - b
```

### `test_calculator.py` (pytest test)

```python
from calculator import add, subtract

def test_add():
    assert add(2, 3) == 5

def test_subtract():
    assert subtract(5, 2) == 3
```

test চালানো:

```bash
pytest
```

> 🔑 **pytest এর নিয়ম:** test ফাইলের নাম `test_` দিয়ে শুরু হতে হয়, আর প্রতিটা test function ও `test_` দিয়ে শুরু হতে হয়। pytest নিজে নিজে এগুলো খুঁজে বের করে চালায়। `assert` দিয়ে যাচাই করা হয় — শর্ত সত্য হলে test pass, না হলে fail।

---

## 🧰 ৫. `unittest` দিয়ে Testing (Python এর built-in tool)

Python এর নিজের ভেতরেই একটা testing library আছে — `unittest`।

### `test_calculator_unittest.py`

```python
import unittest
from calculator import add, subtract

class TestCalculator(unittest.TestCase):
    def test_add(self):
        self.assertEqual(add(2, 3), 5)

    def test_subtract(self):
        self.assertEqual(subtract(5, 2), 3)

if __name__ == "__main__":
    unittest.main()
```

চালানোর command:

```bash
# উপায় ১: unittest module দিয়ে
python3 -m unittest test_calculator_unittest

# উপায় ২: সরাসরি ফাইল চালিয়ে (কারণ শেষে unittest.main() আছে)
python3 test_calculator_unittest.py
```

> ✅ **ভুল ঠিক করা হলো:** তুমি লিখেছিলে `python3 -m test_calculator_unit`। এটা কাজ করবে না দুই কারণে —
> ১) ফাইলের নাম `test_calculator_unittest`, আর
> ২) `-m` এর পরে **module এর নাম** দিতে হয়, তাই এখানে `unittest` লিখতে হবে: `python3 -m unittest test_calculator_unittest`।

### pytest vs unittest — পার্থক্য

| বিষয় | `pytest` | `unittest` |
|------|----------|-----------|
| Install | আলাদা install লাগে (`pip install pytest`) | Python এর সাথেই আসে |
| যাচাই | সাধারণ `assert` | `self.assertEqual()` ইত্যাদি method |
| গঠন | সাধারণ function | `class` এর ভেতরে method |
| লেখা | সহজ ও ছোট | একটু বেশি লিখতে হয় |

---

## ⚙️ ৬. CI (Continuous Integration) — GitHub Actions

**লক্ষ্য:** GitHub এ কোড push করার সাথে সাথেই যেন test **automatically** চলে। এটাকেই বলে **CI (Continuous Integration)**।

### ফোল্ডার গঠন

GitHub Actions এর জন্য একটা নির্দিষ্ট গঠন লাগে:

```
your-project/
├── .github/
│   └── workflows/
│       └── python-tests.yml   ← এই ফাইলে CI এর নিয়ম লেখা থাকে
├── calculator.py
└── test_calculator.py
```

### `python-tests.yml`

```yaml
name: Python CI and tests

on:
  push:
    branches: ["main"]
  pull_request:
    branches: ["main"]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.12"

      - name: Install all the dependencies and packages
        run: |
          python -m pip install --upgrade pip
          pip install pytest flake8

      - name: Lint code with flake8
        run: |
          # Fail on syntax errors or undefined names
          flake8 . --count --select=E9,F63,F7,F82 --show-source --statistics
          # Treat other issues as warnings (doesn't fail build)
          flake8 . --count --exit-zero --max-complexity=10 --max-line-length=88 --statistics

      - name: Run tests with pytest
        run: |
          pytest
```

**এই ফাইল কী বলছে (সহজ ভাষায়):**

| অংশ | মানে |
|-----|------|
| `on: push / pull_request` | `main` branch এ push বা PR হলে চলবে |
| `runs-on: ubuntu-latest` | GitHub একটা Ubuntu কম্পিউটার ভাড়া নিয়ে চালাবে |
| `Checkout code` | তোমার কোড ওই কম্পিউটারে নিয়ে আসবে |
| `Set up Python` | Python 3.12 install করবে |
| `Install dependencies` | pip, pytest, flake8 install করবে |
| `Lint with flake8` | কোডে syntax/style সমস্যা আছে কিনা দেখবে |
| `Run tests with pytest` | সব test চালাবে |

### CI যে কোড টেস্ট করবে

**`calculator.py`** (এই version এ `divide` error raise করে):

```python
def add(a, b):
    return a + b

def subtract(a, b):
    return a - b

def multiply(a, b):
    return a * b

def divide(a, b):
    if b == 0:
        raise ValueError("Cannot divide by zero")
    return a / b
```

**`test_calculator.py`**:

```python
import pytest
from calculator import add, subtract, multiply, divide

def test_add():
    assert add(2, 3) == 5

def test_subtract():
    assert subtract(5, 3) == 2
    assert subtract(0, 5) == -5

def test_multiply():
    assert multiply(2, 3) == 6
    assert multiply(-2, 3) == -6
    assert multiply(0, 5) == 0

def test_divide():
    assert divide(6, 3) == 2
    assert divide(-6, 3) == -2

def test_divide_by_zero():
    with pytest.raises(ValueError):
        divide(5, 0)
```

> 🔑 `with pytest.raises(ValueError):` দিয়ে চেক করা হয় — শূন্য দিয়ে ভাগ করলে সত্যিই `ValueError` আসছে কিনা। error আসাটাই এখানে **সঠিক আচরণ**, তাই এটাও একটা valid test।

---

## 📄 ৭. YAML — বিস্তারিত

### YAML কী?

**YAML** এর পূর্ণরূপ = **YAML Ain't Markup Language** (এটা একটা মজার recursive acronym; আগে বলা হতো "Yet Another Markup Language")।

> ✅ **ভুল ঠিক করা হলো:** তুমি লিখেছিলে "aint markup language" — পুরো ও সঠিক রূপ হলো **YAML Ain't Markup Language**।

এটা মানুষের পড়ার জন্য সহজ (**human-readable**) একটা **data format**। মূলত **configuration (সেটিংস)** লেখার জন্য ব্যবহার হয়। যেমন:
- GitHub Actions / GitLab CI (আমরা উপরে যেটা করলাম)
- Docker Compose
- Kubernetes
- Ansible

ফাইলের extension: `.yml` বা `.yaml`

---

### YAML লেখার নিয়ম (খুব গুরুত্বপূর্ণ)

**১. Key-Value জোড়া** — মূল গঠন `key: value`

```yaml
name: Nure
course: DevOps
age: 25
```

**২. Comment** — `#` দিয়ে

```yaml
# এটা একটা comment
name: Nure   # লাইনের শেষেও comment দেওয়া যায়
```

**৩. Tab চলবে না ❌** — YAML এ **tab কাজ করে না**, শুধু **space** ব্যবহার করতে হয়। এটা YAML এর সবচেয়ে বড় নিয়ম।

**৪. Indentation (ফাঁকা জায়গা)** — nesting বোঝাতে **২টা space** ব্যবহার করা হয় (standard convention)। Indentation এখানে অত্যন্ত **sensitive** — একটু এদিক-ওদিক হলেই error।

```yaml
person:
  name: Nure      # ২ space ভেতরে
  address:
    city: Dhaka   # ৪ space ভেতরে (আরও এক ধাপ nested)
    country: BD
```

**৫. Colon এর পরে space** — `key: value` তে colon `:` এর পরে **অবশ্যই একটা space** দিতে হবে।

```yaml
name: Nure     # ✅ ঠিক (colon এর পর space আছে)
name:Nure      # ❌ ভুল (space নেই)
```

**৬. String ও Quote** — যেগুলো text/string, সেগুলো quote এর ভেতরে রাখা **good practice**।

```yaml
message: "This is a devops course"
```

> ⚠️ একই ফাইলে কোথাও single quote (`'`) আর কোথাও double quote (`"`) মিশিয়ে না দিয়ে, যেকোনো একটাই ধারাবাহিকভাবে ব্যবহার করা ভালো।

---

### YAML List (তালিকা)

`-` (ড্যাশ) দিয়ে list এর প্রতিটা item লেখা হয়:

```yaml
students:
  - rahim
  - ramesh
  - karim
```

---

### Multi-line Text: Folded Block `>` (তোমার screenshot এর উদাহরণ)

কয়েক লাইনের লেখা লিখতে **folded block** ব্যবহার হয়, যা `>` চিহ্ন দিয়ে শুরু হয়। এর কাজ হলো — **প্রতিটা newline (নতুন লাইন) কে একটা space দিয়ে জোড়া লাগিয়ে সব এক লাইনে বানিয়ে দেয়।**

```yaml
# Folded Block:
description: >
  This is a devops course.
  Nure is shaming.
  Worst course ever.
```

**Output** (সব এক লাইনে, newline গুলো space হয়ে গেছে):

```
This is a devops course. Nure is shaming. Worst course ever.
```

### বোনাস: Literal Block `|` (তুলনার জন্য)

আরেকটা style আছে — **literal block** `|` — যেটা লাইন-ভাঙা (line break) **যেমন আছে তেমনই রাখে**:

```yaml
description: |
  This is a devops course.
  Nure is shaming.
  Worst course ever.
```

**Output** (আলাদা আলাদা লাইনেই থাকে):

```
This is a devops course.
Nure is shaming.
Worst course ever.
```

> 🔑 মনে রাখার সহজ উপায়: `>` (folded) = **এক লাইন** বানায়, `|` (literal, দেখতে সোজা দাঁড়ি) = **লাইন যেমন আছে তেমন** রাখে।

---

### YAML ফাইল যাচাই করা: `yamllint`

YAML ফাইল ঠিকভাবে লেখা হয়েছে কিনা (indentation, space ইত্যাদি) তা check করার জন্য **`yamllint`** package ব্যবহার করা হয়।

```bash
pip3 install yamllint
yamllint python-tests.yml
```

---

## 📝 আজকের ক্লাসের সারাংশ

- প্রতিটা `.py` ফাইল একটা **module**; `import` দিয়ে এক ফাইল থেকে আরেক ফাইলে কোড আনা যায় (দরকারিটাই import করা best)।
- **Virtual environment** (`python3 -m venv`) দিয়ে প্রতি প্রজেক্টের package আলাদা রাখা যায়।
- **Unit testing:** `pytest` (সহজ, `assert`) আর `unittest` (built-in, `class` + `assertEqual`)।
- **CI (GitHub Actions):** `.github/workflows/*.yml` ফাইলে নিয়ম লিখলে push এর সাথে test **automatically** চলে।
- **YAML** = *YAML Ain't Markup Language*; space দিয়ে indentation (tab নয়), colon এর পর space, `>` folded আর `|` literal block।

---

*Happy Coding & Testing! 🚀*