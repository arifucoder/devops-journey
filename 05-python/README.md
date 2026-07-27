# 🐍 Python শেখার নোটস — সম্পূর্ণ গাইড

> এই README এ Python এর প্রথম তিনটি ক্লাসের সব নোট একসাথে গোছানো আছে। Basic (variable, data type) থেকে শুরু করে Module, Unit Testing, CI/CD আর YAML পর্যন্ত — ধাপে ধাপে সাজানো।

---

## 📚 সূচিপত্র (Table of Contents)

### [Class 1 — Python Basics](#-class-1--python-basics)
- [Python Install করা](#-১-python-install-করা-macos--homebrew-দিয়ে)
- [Variable](#-২-variable-ভেরিয়েবল)
- [Data Types](#-৩-python-এর-data-types-ডাটা-টাইপ)
- [Comment করা](#-৪-comment-করা)
- [Data Type চেক করা](#-৫-data-type-চেক-করা)
- [Type Casting](#-৬-type-casting-এক-টাইপ-থেকে-আরেক-টাইপে-বদলানো)
- [Formatted Printing](#-৭-formatted-printing-সুন্দর-করে-print-করা)

### [Class 2 — Data Structures ও Logic](#-class-2--data-structures-ও-logic)
- [Dictionary](#-১-dictionary-ডিকশনারি)
- [List এর Functions](#-২-list-এর-default-functions)
- [Loop](#-৩-loop-লুপ)
- [List Comprehension](#-৪-list-comprehension)
- [Arithmetic Operators](#-৫-arithmetic-operators-গাণিতিক-অপারেটর)
- [Comparison Operators](#-৬-comparison-operators-তুলনার-অপারেটর)
- [Logical Operators](#-৭-logical-operators-and-or)
- [Conditions (if/elif/else)](#-৮-conditions-if--elif--else)
- [Functions](#️-৯-functions-ফাংশন)

### [Class 3 — Module, Testing, CI ও YAML](#-class-3--module-unit-testing-ci-ও-yaml)
- [Python Module System](#-১-python-module-system)
- [Testing কেন দরকার](#-২-testing-কেন-দরকার)
- [Virtual Environment](#-৩-virtual-environment-venv)
- [pytest দিয়ে Testing](#-৪-pytest-দিয়ে-unit-testing)
- [unittest দিয়ে Testing](#-৫-unittest-দিয়ে-testing-python-এর-built-in-tool)
- [CI — GitHub Actions](#️-৬-ci-continuous-integration--github-actions)
- [YAML](#-৭-yaml--বিস্তারিত)

---
---

# 🟢 Class 1 — Python Basics

> Python শেখার প্রথম ক্লাসের গোছানো নোট। এখানে installation থেকে শুরু করে variable, data types, type casting আর formatted printing পর্যন্ত সব আছে।

---

## 📦 ১. Python Install করা (macOS — Homebrew দিয়ে)

macOS এ `brew` (Homebrew) হলো একটা package manager, যেটার মাধ্যমে সহজে software install করা যায়।

```bash
# brew কে update করে নেওয়া (নতুন সব info আনার জন্য)
brew update

# Python install করা
brew install python

# Python এর version দেখা
python3 --version

# pip (Python এর package installer) এর version দেখা
pip3 --version

# আবার update দিয়ে Python কে সর্বশেষ version এ নিয়ে যাওয়া
brew update
brew upgrade python
```

> 💡 **টিপস:** `--version` দিয়ে চেক করলে বুঝা যায় install ঠিকভাবে হয়েছে কিনা। version number দেখালেই সব ঠিক আছে।

---

## 🏷️ ২. Variable (ভেরিয়েবল)

**Variable** হলো এমন একটা নাম, যার ভেতরে আমরা কোনো value জমা রাখি।

সহজ উদাহরণ: 🍼 যখন কোনো শিশু জন্ম নেয়, তখন তাকে একটা নাম দেওয়া হয় — সেই নামটাই হলো **variable**। আর শিশুটা নিজেই হলো সেই নামের ভেতরে থাকা **value**।

```python
name = "Rahim"   # name হলো variable, "Rahim" হলো তার value
age = 12         # age হলো variable, 12 হলো তার value
```

---

## 🧩 ৩. Python এর Data Types (ডাটা টাইপ)

Python এ প্রধান কয়েকটা data type হলো:

| Data Type | কী রাখে | উদাহরণ |
|-----------|---------|--------|
| `str` | লেখা / text (string) | `"Hello"` |
| `int` | পূর্ণ সংখ্যা (integer) | `25` |
| `float` | দশমিক সংখ্যা | `3.14` |
| `bool` | সত্য / মিথ্যা | `True`, `False` |
| `list` | একাধিক value এর তালিকা | `[1, 2, 3]` |

### ৩.১ String (`str`)

String মানে **লেখা বা text**। নিচের যেকোনো একটা quote দিয়ে string লেখা যায়:

```python
a = 'single quote'
b = "double quote"
c = """triple double quote"""
d = '''triple single quote'''
```

সবগুলোই string হিসেবে variable এ রাখা যায়।

> ✅ **মনে রাখো:** Triple quotes (`'''` বা `"""`) এর আসল কাজ হলো **একাধিক লাইনের (multi-line) লেখা** একসাথে রাখা।

```python
paragraph = """এটা প্রথম লাইন
এটা দ্বিতীয় লাইন
এটা তৃতীয় লাইন"""

print(paragraph)
```

### ৩.২ Integer (`int`) ও Float (`float`)

```python
x = 25          # int → পূর্ণ সংখ্যা
price = 99.50   # float → দশমিক সংখ্যা
```

### ৩.৩ Boolean (`bool`)

শুধু দুইটা value থাকে: `True` অথবা `False`।

```python
is_student = True
is_teacher = False
```

### ৩.৪ List

একাধিক value একসাথে একটা তালিকায় রাখার জন্য `list` ব্যবহার হয়। `[ ]` bracket দিয়ে লেখা হয়।

```python
fruits = ["আম", "কাঁঠাল", "লিচু"]
numbers = [10, 20, 30, 40]
```

---

## 💬 ৪. Comment করা

Code এর ভেতরে নিজের জন্য নোট বা ব্যাখ্যা লিখতে **comment** ব্যবহার হয়। Python এটা run করে না, শুধু আমরা পড়ার জন্য রাখি।

```python
# এটা একটা comment — Python এটা ignore করবে
name = "Karim"   # এভাবে লাইনের শেষেও comment লেখা যায়
```

---

## 🔍 ৫. Data Type চেক করা

কোনো variable এর data type জানতে হলে `type()` ব্যবহার করি:

```python
print(type(name))        # <class 'str'>
print(type(age))         # <class 'int'>
print(type(price))       # <class 'float'>
print(type(is_student))  # <class 'bool'>
```

> নিয়ম: `print(type(variable_name))`

---

## 🔄 ৬. Type Casting (এক টাইপ থেকে আরেক টাইপে বদলানো)

কখনো কখনো একটা data type কে অন্য type এ বদলাতে হয়। যেমন — string এ থাকা সংখ্যাকে int বানানো।

```python
x = 2
y = '3'          # এটা string, সংখ্যা নয়!

result = x + int(y)   # int(y) দিয়ে '3' কে সংখ্যা 3 বানালাম
print(result)         # 5
```

> ⚠️ যদি `int(y)` না লিখতাম, তাহলে error আসত — কারণ `int + str` যোগ করা যায় না।

---

## 🖨️ ৭. Formatted Printing (সুন্দর করে print করা)

Variable এর value কে বাক্যের ভেতরে বসিয়ে print করার দুইটা জনপ্রিয় নিয়ম আছে।

### উপায় ১: f-string (সবচেয়ে সহজ)

```python
name = "Rahim"
age = 12

print(f"Your name is {name} and your age is {age}")
# Output: Your name is Rahim and your age is 12
```

### উপায় ২: `.format()` method

```python
print("Your name is {} and your age is {}".format(name, age))
# Output: Your name is Rahim and your age is 12
```

---

## 📝 Class 1 সারাংশ

- `brew` দিয়ে macOS এ Python install করা যায়।
- **Variable** = value রাখার নাম (শিশুর নামের মতো)।
- প্রধান data types: `str`, `int`, `float`, `bool`, `list`।
- `#` দিয়ে **comment**, `type()` দিয়ে data type চেক।
- `int()`, `str()` দিয়ে **type casting**।
- `f-string` আর `.format()` দিয়ে সুন্দর করে print।

---
---

# 🔵 Class 2 — Data Structures ও Logic

> এই ক্লাসে আছে: Dictionary, List এর function, Loop, List Comprehension, Arithmetic operator, Comparison operator, Logical operator, Conditions (if/elif/else) আর Functions।

---

## 📖 ১. Dictionary (ডিকশনারি)

Dictionary হলো এমন একটা data type যেখানে data **key-value pair** আকারে থাকে, আর পুরোটা **curly braces `{ }`** দিয়ে ঘেরা থাকে।

```python
# Creating the dictionary
employee_directory = {
    "EMP101": "Anis Rahman",
    "EMP102": "Sultana Kamal",
    "EMP103": "Tamim Iqbal"
}
```

এখানে `"EMP101"` হলো **key** আর `"Anis Rahman"` হলো তার **value**।

> ✅ **মনে রাখো:**
> - **Key** যেকোনো **immutable (অপরিবর্তনযোগ্য)** type হতে পারে — যেমন `str`, `int`, `float`, `tuple`, `bool`। তাই key শুধু string ই হয় তা নয়, সংখ্যাও হতে পারে (যেমন `{1: "one", 2: "two"}`)।
> - **Value** যেকোনো type এর data হতে পারে (string, number, list, এমনকি আরেকটা dictionary!)।

### Dictionary এর কাজগুলো

```python
# 1. Looking up a value (দ্রুত খুঁজে বের করা)
print(employee_directory["EMP102"])   # Sultana Kamal

# 2. Adding a new employee (নতুন key-value যোগ করা)
employee_directory["EMP104"] = "Sakib Al Hasan"

# 3. Overwriting/Updating an existing key (পুরনো value বদলে দেওয়া)
employee_directory["EMP101"] = "Khalilur Hasan"

print(employee_directory)
```

> 💡 একই key আবার লিখে নতুন value দিলে পুরনোটা **overwrite** হয়ে যায়, নতুন কিছু যোগ হয় না।

---

## 📋 ২. List এর Default Functions

List এ একাধিক জিনিস একসাথে রাখা যায়, আর প্রতিটা জিনিসের একটা **position (index)** থাকে।

```python
grocery_list = ["Eggs", "Milk", "Bread"]
```

### Index দিয়ে item এ পৌঁছানো

Index শুরু হয় **0** থেকে (1 থেকে নয়!)।

```python
print(grocery_list[0])   # Eggs  (প্রথম item)
print(grocery_list[1])   # Milk  (দ্বিতীয় item)
print(grocery_list[-1])  # Bread (একদম শেষের item — negative হলে শেষ থেকে গোনা)
```

### item যোগ করা

```python
# append() — একদম শেষে item যোগ করে
grocery_list.append("Apples")

# insert() — নির্দিষ্ট position এ item ঢোকায়  →  list.insert(position, item)
grocery_list.insert(1, "Butter")
```

### item সরানো / মুছে ফেলা

```python
# remove() — নাম ধরে item মুছে দেয়
grocery_list.remove("Milk")

# pop() — position number দিয়ে item সরায় এবং সরানো জিনিসটা ফেরত দেয়
removed_item = grocery_list.pop(2)
print(f"I took this out of the list: {removed_item}")
```

> 🔑 পার্থক্য: `remove()` **নাম** দিয়ে মুছে, আর `pop()` **position** দিয়ে সরায় ও সরানো জিনিসটা **ফেরত (return)** দেয়।

### খোঁজা, গোনা আর সাজানো

```python
if "Eggs" in grocery_list:          # Searching (in দিয়ে চেক)
    print("Yes, you need to buy eggs.")

print(len(grocery_list))            # Counting (মোট সংখ্যা)

grocery_list.sort()                 # Sorting (A→Z ক্রমে সাজানো)
print(grocery_list)
```

---

## 🔁 ৩. Loop (লুপ)

একই কাজ বারবার করার জন্য loop ব্যবহার হয়।

```python
dollar_prices = [1.50, 4.99, 10.00, 25.50]
cents_prices = []                      # ফলাফল রাখার খালি list

for price in dollar_prices:
    cents = int(price * 100)
    cents_prices.append(cents)

print(cents_prices)                    # [150, 499, 1000, 2550]
```

---

## ⚡ ৪. List Comprehension

উপরের loop কে **এক লাইনে** লেখার সহজ উপায়:

```python
dollar_prices = [1.50, 4.99, 10.00, 25.50]
cents_prices = [int(price * 100) for price in dollar_prices]
print(cents_prices)                    # [150, 499, 1000, 2550]
```

> 💡 গঠন: `[কাজ for item in list]`

---

## ➕ ৫. Arithmetic Operators (গাণিতিক অপারেটর)

| Operator | কাজ | উদাহরণ | ফলাফল |
|----------|-----|--------|-------|
| `*` | গুণ | `3 * 15` | `45` |
| `+` | যোগ | `45 + 4` | `49` |
| `-` | বিয়োগ | `49 - 5` | `44` |
| `/` | ভাগ (float দেয়) | `44 / 4` | `11.0` |
| `//` | floor division (গোটা অংশ) | `24 // 5` | `4` |
| `%` | modulo (ভাগশেষ) | `24 % 5` | `4` |
| `**` | ঘাত/power | `12 ** 2` | `144` |

```python
# উদাহরণ (pizza party!)
pizza_count = 3
price_per_pizza = 15
total = pizza_count * price_per_pizza      # * গুণ → 45

total_slices = 24
people = 5
print(total_slices // people)              # // গোটা অংশ → 4
print(total_slices % people)               # %  ভাগশেষ → 4
print(12 ** 2)                             # ** power → 144
```

---

## ⚖️ ৬. Comparison Operators (তুলনার অপারেটর)

এগুলো দুইটা value তুলনা করে **`True` বা `False`** ফেরত দেয়।

```python
movie_ticket_price = 12
minimum_age = 18
customer_age = 20
customer_cash = 12

print(customer_age > minimum_age)          # True   (>  বড়)
print(movie_ticket_price < customer_cash)  # False  (<  ছোট)
print(customer_cash == movie_ticket_price) # True   (== সমান কিনা)
print(customer_age != minimum_age)         # True   (!= সমান নয় কিনা)
print(customer_cash >= movie_ticket_price) # True   (>= বড় বা সমান)
print(customer_age <= 17)                  # False  (<= ছোট বা সমান)
```

> ⚠️ **খুব গুরুত্বপূর্ণ:** তুলনা করতে **দুইটা** equal sign `==` লাগে। একটা `=` দিয়ে value **assign** করা হয়, তুলনা নয়।

---

## 🔗 ৭. Logical Operators (`and`, `or`)

| Operator | কখন True হয় |
|----------|-------------|
| `and` | **সব** শর্ত True হলে |
| `or` | **অন্তত একটা** শর্ত True হলে |

```python
# and: দুই দিকের শর্তই True হতে হবে
not_married = True
age = 20
if not_married == True and age >= 18:
    print("Most eligible bachelor getting married")
else:
    print("Papa will....")

# or: অন্তত একটা শর্ত True হলেই যথেষ্ট
has_job = True
if has_job == True or age >= 18:
    print("Most eligible bachelor getting married")
else:
    print("Okormar dheki, bia korte asche, maira...")
```

---

## 🚦 ৮. Conditions (if / elif / else)

**Condition** এর মূলে আছে **Boolean (`bool`)** — যার মান শুধু `True`/`False`। এই Boolean দিয়েই program এর গতিপথ নিয়ন্ত্রণ করা হয়।

```python
# Movie Ticket Pricing
age = 25

if age < 4:
    print("Ticket is free for toddlers!")
elif age <= 12:
    print("Child ticket price: $8.00")
elif age < 65:
    print("Adult ticket price: $15.00")
else:
    print("Senior citizen ticket price: $10.00")
```

> 🔑 Python উপর থেকে নিচে চেক করে। প্রথম যে শর্ত `True` হয়, শুধু সেটার block চলে, বাকিগুলো skip হয়। `age = 25` হওয়ায় → "Adult ticket price: $15.00"।

---

## 🛠️ ৯. Functions (ফাংশন)

Function হলো একটা কাজের logic একবার লিখে বারবার ব্যবহার করার উপায়। `def` keyword দিয়ে তৈরি।

```python
def insert_user_in_database(name, age):
    print(f"User inserted in DB for {name}, {age}")

insert_user_in_database("Alice", 34)   # function কে call করতে হয়
```

### `return` — value ফেরত দেওয়া

`print()` শুধু দেখায়, কিন্তু `return` দিয়ে value ফেরত পাওয়া যায় যা পরে ব্যবহার করা যায়।

```python
def insert_user_in_database(name, age):
    return f"User inserted in DB for {name}, {age}"

print(insert_user_in_database("Alice", 34))
```

### কেন Function দরকার? (DRY principle)

একই logic বারবার না লিখে, একবার function বানিয়ে বারবার reuse করা যায়:

```python
def calculate_tax(amount, state):
    if state == "TX":
        tax_rate = 0.0625
    elif state == "CA":
        tax_rate = 0.0725
    else:
        tax_rate = 0.05
    return amount * tax_rate

print(f"Customer 1 Tax: ${calculate_tax(1000, 'TX')}")
print(f"Customer 2 Tax: ${calculate_tax(2500, 'CA')}")
print(f"Customer 3 Tax: ${calculate_tax(500, 'NY')}")
```

> 💡 এটাই **DRY principle** — *Don't Repeat Yourself*।

---

## 📝 Class 2 সারাংশ

- **Dictionary** = `{key: value}` জোড়া। Key immutable, value যেকোনো type।
- **List** এর কাজ: `append()`, `insert()`, `remove()`, `pop()`, `len()`, `sort()`, `in`।
- **Loop** দিয়ে বারবার কাজ; **List Comprehension** সেটাকে এক লাইনে।
- **Arithmetic:** `+ - * / // % **` | **Comparison:** `> < == != >= <=`
- **Logical:** `and` (সব True), `or` (একটা True হলেই)।
- **Conditions:** `if / elif / else` | **Functions:** `def` + `return` (DRY)।

---
---

# 🟣 Class 3 — Module, Unit Testing, CI ও YAML

> এই ক্লাসে আছে: Python Module (import system), Virtual Environment, Unit Testing (pytest ও unittest), GitHub Actions দিয়ে CI/CD, আর YAML ফাইল।

---

## 📦 ১. Python Module System

Python **modular system** এ কাজ করে — কোডকে ছোট ছোট আলাদা ফাইলে ভাগ করে দরকার মতো এক ফাইল থেকে আরেক ফাইলে import করা যায়। প্রতিটা `.py` ফাইল আসলে একটা **module**।

### ধাপ ১: একটা module তৈরি → `pymodules.py`

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

### ধাপ ২: অন্য ফাইলে module ব্যবহার → `app.py`

```python
# উপায় ১: পুরো module import (সব function আসে)
import pymodules
print(pymodules.add(2, 3))

# উপায় ২: শুধু দরকারি function import + নতুন নাম (alias)
from pymodules import add as joge
print(joge(2, 3))
```

> 💡 **মূল কথা:**
> - `import pymodules` → ওই ফাইলের **সবকিছু** আসে (`pymodules.add()` লিখতে হয়)।
> - `from pymodules import add` → শুধু **দরকারিটাই** আসে (সরাসরি `add()`)।
> - **Best practice:** যা প্রয়োজন শুধু সেটাই import করা।

---

## 🧪 ২. Testing কেন দরকার?

Professional developer রা কোড ঠিকভাবে কাজ করছে কিনা যাচাই করতে **test** লেখেন। সবচেয়ে বেসিক ও জনপ্রিয় হলো **unit testing** — কোডের ছোট ছোট অংশ (যেমন প্রতিটা function) আলাদাভাবে পরীক্ষা করা।

**Industry তে কাজের ধরন:**
1. Developer নিজের computer এ test লিখে run করেন।
2. তারপর কোড GitHub / GitLab এ push করেন।
3. Push করার সাথে সাথে test **automatically** চলে (এটাই CI)।

---

## 🌐 ৩. Virtual Environment (venv)

বিভিন্ন প্রজেক্টে বিভিন্ন version এর package লাগতে পারে। এক প্রজেক্টের package যেন আরেকটার সাথে সংঘর্ষ (conflict) না করে, তার জন্য আলাদা **virtual environment** বানানো হয়।

```bash
# 1. তৈরি করা (.venv বা যেকোনো নাম)
python3 -m venv .venv

# 2. Activate করা
source .venv/bin/activate

# 3. দরকারি package install
pip3 install pytest

# 4. কাজ শেষে deactivate
deactivate
```

> 💡 activate করার পর যা install করবে সব ওই environment এর ভেতরেই থাকবে — মূল system পরিষ্কার থাকবে।

---

## ✅ ৪. pytest দিয়ে Unit Testing

সবচেয়ে সহজ ও জনপ্রিয় testing tool হলো **pytest**।

```python
# calculator.py
def add(a, b):
    return a + b

def subtract(a, b):
    return a - b
```

```python
# test_calculator.py
from calculator import add, subtract

def test_add():
    assert add(2, 3) == 5

def test_subtract():
    assert subtract(5, 2) == 3
```

```bash
pytest   # test চালানো
```

> 🔑 **নিয়ম:** test ফাইল ও test function এর নাম `test_` দিয়ে শুরু হতে হয়; pytest নিজে খুঁজে চালায়। `assert` সত্য হলে pass, না হলে fail।

---

## 🧰 ৫. `unittest` দিয়ে Testing (Python এর built-in tool)

```python
# test_calculator_unittest.py
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

```bash
# উপায় ১: unittest module দিয়ে
python3 -m unittest test_calculator_unittest

# উপায় ২: সরাসরি ফাইল চালিয়ে (কারণ শেষে unittest.main() আছে)
python3 test_calculator_unittest.py
```

### pytest vs unittest

| বিষয় | `pytest` | `unittest` |
|------|----------|-----------|
| Install | আলাদা লাগে (`pip install pytest`) | Python এর সাথেই আসে |
| যাচাই | সাধারণ `assert` | `self.assertEqual()` ইত্যাদি |
| গঠন | সাধারণ function | `class` এর ভেতরে method |
| লেখা | সহজ ও ছোট | একটু বেশি লিখতে হয় |

---

## ⚙️ ৬. CI (Continuous Integration) — GitHub Actions

**লক্ষ্য:** GitHub এ কোড push করার সাথে সাথেই test যেন **automatically** চলে।

### ফোল্ডার গঠন

```
your-project/
├── .github/
│   └── workflows/
│       └── python-tests.yml   ← এই ফাইলে CI এর নিয়ম
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
| `Checkout code` | তোমার কোড ওই কম্পিউটারে আনবে |
| `Set up Python` | Python 3.12 install করবে |
| `Install dependencies` | pip, pytest, flake8 install করবে |
| `Lint with flake8` | কোডে syntax/style সমস্যা আছে কিনা দেখবে |
| `Run tests with pytest` | সব test চালাবে |

### CI যে কোড টেস্ট করবে (উন্নত version)

```python
# calculator.py — এই version এ divide error raise করে
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

```python
# test_calculator.py
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

> 🔑 `with pytest.raises(ValueError):` দিয়ে চেক হয় — শূন্য দিয়ে ভাগ করলে সত্যিই `ValueError` আসছে কিনা। error আসাটাই এখানে **সঠিক আচরণ**।

---

## 📄 ৭. YAML — বিস্তারিত

### YAML কী?

**YAML** এর পূর্ণরূপ = **YAML Ain't Markup Language** (মজার recursive acronym; আগে বলা হতো "Yet Another Markup Language")।

এটা মানুষের পড়ার জন্য সহজ (**human-readable**) একটা **data format**, মূলত **configuration (সেটিংস)** লেখার জন্য। যেমন — GitHub Actions, Docker Compose, Kubernetes, Ansible। ফাইলের extension: `.yml` বা `.yaml`।

### YAML লেখার নিয়ম (গুরুত্বপূর্ণ)

**১. Key-Value জোড়া** — মূল গঠন `key: value`

```yaml
name: Nure
course: DevOps
age: 25
```

**২. Comment** — `#` দিয়ে

```yaml
# এটা একটা comment
name: Nure   # লাইনের শেষেও comment
```

**৩. Tab চলবে না ❌** — YAML এ **tab কাজ করে না**, শুধু **space**। এটাই সবচেয়ে বড় নিয়ম।

**৪. Indentation** — nesting বোঝাতে **২টা space**। এটা খুবই **sensitive** — একটু এদিক-ওদিক হলেই error।

```yaml
person:
  name: Nure      # ২ space ভেতরে
  address:
    city: Dhaka   # ৪ space ভেতরে (আরও এক ধাপ nested)
    country: BD
```

**৫. Colon এর পরে space** — `key: value` তে colon `:` এর পরে **অবশ্যই একটা space**।

```yaml
name: Nure     # ✅ ঠিক
name:Nure      # ❌ ভুল (space নেই)
```

**৬. String ও Quote** — string গুলো quote এ রাখা **good practice**। একই ফাইলে single ও double quote মিশিয়ে না দিয়ে যেকোনো একটাই ধারাবাহিকভাবে ব্যবহার করা ভালো।

```yaml
message: "This is a devops course"
```

### YAML List (তালিকা)

`-` (ড্যাশ) দিয়ে প্রতিটা item:

```yaml
students:
  - rahim
  - ramesh
  - karim
```

### Multi-line Text: Folded `>` vs Literal `|`

**Folded block (`>`)** — newline গুলোকে space দিয়ে জোড়া লাগিয়ে **এক লাইন** বানায়:

```yaml
description: >
  This is a devops course.
  Nure is shaming.
  Worst course ever.
```

**Output** (সব এক লাইনে):

```text
This is a devops course. Nure is shaming. Worst course ever.
```

**Literal block (`|`)** — লাইন-ভাঙা **যেমন আছে তেমনই রাখে**:

```yaml
description: |
  This is a devops course.
  Nure is shaming.
  Worst course ever.
```

**Output** (আলাদা আলাদা লাইনেই থাকে):

```text
This is a devops course.
Nure is shaming.
Worst course ever.
```

> 🔑 মনে রাখার সহজ উপায়: `>` (folded) = **এক লাইন**; `|` (literal, দেখতে সোজা দাঁড়ি) = **লাইন যেমন আছে তেমন**।

### YAML ফাইল যাচাই: `yamllint`

```bash
pip3 install yamllint
yamllint python-tests.yml
```

---

## 📝 Class 3 সারাংশ

- প্রতিটা `.py` ফাইল একটা **module**; `import` দিয়ে কোড আনা যায় (দরকারিটাই import best)।
- **Virtual environment** (`python3 -m venv`) দিয়ে package আলাদা রাখা।
- **Unit testing:** `pytest` (সহজ, `assert`) আর `unittest` (built-in, `class` + `assertEqual`)।
- **CI (GitHub Actions):** `.github/workflows/*.yml` — push এর সাথে test **automatically** চলে।
- **YAML** = *YAML Ain't Markup Language*; space দিয়ে indentation (tab নয়), colon এর পর space, `>` folded আর `|` literal।

---

**Happy Coding & Testing! 🚀**

*Python Basics → Data Structures → Testing & CI/CD*