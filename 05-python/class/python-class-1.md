# 🐍 Python Class 1 — নোটস

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

---

### ৩.১ String (`str`)

String মানে **লেখা বা text**। নিচের যেকোনো একটা quote দিয়ে string লেখা যায়:

```python
a = 'single quote'
b = "double quote"
c = """triple double quote"""
d = '''triple single quote'''
```

সবগুলোই string হিসেবে variable এ রাখা যায়।

> ✅ **ঠিক তথ্য:** Triple quotes (`'''` বা `"""`) এর আসল কাজ হলো **একাধিক লাইনের (multi-line) লেখা** একসাথে রাখা। অর্থাৎ যে লেখা কয়েক লাইন জুড়ে থাকবে, সেটা triple quote দিয়ে লিখতে হয়।

```python
paragraph = """এটা প্রথম লাইন
এটা দ্বিতীয় লাইন
এটা তৃতীয় লাইন"""

print(paragraph)
```

---

### ৩.২ Integer (`int`) ও Float (`float`)

```python
x = 25          # int → পূর্ণ সংখ্যা
price = 99.50   # float → দশমিক সংখ্যা
```

---

### ৩.৩ Boolean (`bool`)

শুধু দুইটা value থাকে: `True` অথবা `False`।

```python
is_student = True
is_teacher = False
```

---

### ৩.৪ List

একাধিক value একসাথে একটা তালিকায় রাখার জন্য `list` ব্যবহার হয়। `[ ]` bracket দিয়ে লেখা হয়।

```python
fruits = ["আম", "কাঁঠাল", "লিচু"]
numbers = [10, 20, 30, 40]
```

---

## 💬 ৪. Comment করা

Code এর ভেতরে নিজের জন্য নোট বা ব্যাখ্যা লিখতে **comment** ব্যবহার হয়। Python এটা run করে না, শুধু আমরা পড়ার জন্য রাখি।

`#` চিহ্ন দিয়ে comment লেখা হয়:

```python
# এটা একটা comment — Python এটা ignore করবে
name = "Karim"   # এভাবে লাইনের শেষেও comment লেখা যায়
```

---

## 🔍 ৫. Data Type চেক করা

কোনো variable এর data type জানতে হলে `type()` ব্যবহার করি:

```python
print(type(name))     # <class 'str'>
print(type(age))      # <class 'int'>
print(type(price))    # <class 'float'>
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
```

**Output:**

```
Your name is Rahim and your age is 12
```

### উপায় ২: `.format()` method

```python
print("Your name is {} and your age is {}".format(name, age))
```

**Output:**

```
Your name is Rahim and your age is 12
```

> ✅ **ভুল ঠিক করা হলো:** তোমার নোটে `.format()` এর quote জায়গা মতো ছিল না। সঠিক নিয়ম — লেখার string টা quote দিয়ে বন্ধ করে **তারপর** `.format(...)` লিখতে হয়:
> `"... {} ... {}".format(value1, value2)`

---

## 📝 আজকের ক্লাসের সারাংশ

- `brew` দিয়ে macOS এ Python install করা যায়।
- **Variable** = value রাখার নাম (শিশুর নামের মতো)।
- প্রধান data types: `str`, `int`, `float`, `bool`, `list`।
- `#` দিয়ে **comment** করা হয়।
- `type()` দিয়ে data type চেক করা হয়।
- `int()`, `str()` দিয়ে **type casting** করা হয়।
- `f-string` আর `.format()` দিয়ে সুন্দর করে print করা যায়।

---

*Happy Coding! 🚀*