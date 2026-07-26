# 🐍 Python Class 2 — নোটস

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

> ✅ **ভুল ঠিক করা হলো:** তুমি লিখেছিলে "key always string thake" — এটা পুরোপুরি ঠিক নয়। আসল নিয়ম হলো:
> - **Key** যেকোনো **immutable (অপরিবর্তনযোগ্য)** type হতে পারে — যেমন `str`, `int`, `float`, `tuple`, `bool`। তাই key শুধু string ই হয় তা নয়, সংখ্যাও হতে পারে (যেমন `{1: "one", 2: "two"}`)।
> - **Value** যেকোনো type এর data হতে পারে (string, number, list, এমনকি আরেকটা dictionary!)।
>
> উপরের উদাহরণে key গুলো string, তাই quote দিয়ে লেখা হয়েছে। কিন্তু key সবসময় string হতে হবে এমন কোনো নিয়ম নেই।

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
# Creating our shopping list
grocery_list = ["Eggs", "Milk", "Bread"]
```

### Index দিয়ে item এ পৌঁছানো

Index শুরু হয় **0** থেকে (1 থেকে নয়!)।

```python
print(grocery_list[0])   # Eggs  (প্রথম item)
print(grocery_list[1])   # Milk  (দ্বিতীয় item)

# Negative number দিলে শেষ থেকে গোনা শুরু হয়
print(grocery_list[-1])  # Bread (একদম শেষের item)
```

### item যোগ করা

```python
# 1. append() — একদম শেষে item যোগ করে
grocery_list.append("Apples")
print(grocery_list)

# 2. insert() — নির্দিষ্ট position এ item ঢোকায়
# Syntax: list.insert(position, item)
grocery_list.insert(1, "Butter")
print(grocery_list)
```

### item সরানো / মুছে ফেলা

```python
# 1. remove() — নাম ধরে item মুছে দেয়
grocery_list.remove("Milk")
print(grocery_list)

# 2. pop() — position number দিয়ে item সরায়
# position 2 এর item সরিয়ে সেটা একটা variable এ জমা রাখে
removed_item = grocery_list.pop(2)
print(f"I took this out of the list: {removed_item}")
print(grocery_list)
```

> 🔑 পার্থক্য: `remove()` **নাম** দিয়ে মুছে, আর `pop()` **position** দিয়ে সরায় এবং সরানো জিনিসটা **ফেরত (return)** দেয়।

### খোঁজা, গোনা আর সাজানো

```python
# Searching (in দিয়ে চেক করা)
if "Eggs" in grocery_list:
    print("Yes, you need to buy eggs.")

# Counting (len() দিয়ে মোট সংখ্যা)
print(len(grocery_list))

# Sorting (sort() দিয়ে A→Z ক্রমে সাজানো)
grocery_list.sort()
print(grocery_list)
# Output: ['Apples', 'Butter', 'Eggs']  (এখন A-Z order এ!)
```

---

## 🔁 ৩. Loop (লুপ)

একই কাজ বারবার করার জন্য loop ব্যবহার হয়। নিচে প্রতিটা দামকে ১০০ দিয়ে গুণ করে cents এ বদলানো হচ্ছে:

```python
# A list of product prices in dollars
dollar_prices = [1.50, 4.99, 10.00, 25.50]

# ফলাফল রাখার জন্য একটা খালি list
cents_prices = []

# প্রতিটা দাম নিয়ে 100 দিয়ে গুণ করে নতুন list এ যোগ করা
for price in dollar_prices:
    cents = int(price * 100)
    cents_prices.append(cents)

print(cents_prices)
```

---

## ⚡ ৪. List Comprehension

উপরের ৪ লাইনের loop কে **এক লাইনে** লেখার সহজ উপায়:

```python
dollar_prices = [1.50, 4.99, 10.00, 25.50]

# এই এক লাইন উপরের পুরো loop এর সমান কাজ করে
cents_prices = [int(price * 100) for price in dollar_prices]

print(cents_prices)
# Output: [150, 499, 1000, 2550]
```

> 💡 List comprehension এর গঠন: `[কাজ for item in list]`

---

## ➕ ৫. Arithmetic Operators (গাণিতিক অপারেটর)

```python
# --- 1. Multiplication (*) ---
# 3টা pizza, প্রতিটার দাম $15
pizza_count = 3
price_per_pizza = 15
total_pizza_cost = pizza_count * price_per_pizza
print(f"Total pizza cost: ${total_pizza_cost}")   # 45

# --- 2. Addition (+) ---
# সাথে $4 এর soda যোগ
soda_cost = 4
total_bill = total_pizza_cost + soda_cost
print(f"Total bill with soda: ${total_bill}")     # 49

# --- 3. Subtraction (-) ---
# $5 ছাড়ের coupon
coupon = 5
final_bill = total_bill - coupon
print(f"Final bill after coupon: ${final_bill}")  # 44

# --- 4. Division (/) ---
# 4 বন্ধুর মধ্যে সমান ভাগ
# নোট: সাধারণ ভাগ সবসময় float (দশমিক সংখ্যা) দেয়
friends = 4
cost_per_person = final_bill / friends
print(f"Each person pays: ${cost_per_person}")    # 11.0

# --- 5. Floor Division (//) ---
# মোট slice = 3 * 8 = 24. 5 জনের মধ্যে সমান কয়টা GOTA slice পায়?
total_slices = 24
people_eating = 5
slices_per_person = total_slices // people_eating
print(f"Whole slices per person: {slices_per_person}")  # 4

# --- 6. Modulo / Mod (%) ---
# সবাই সমান ভাগ নেওয়ার পর box এ কয়টা slice বেঁচে থাকবে
leftover_slices = total_slices % people_eating
print(f"Leftover slices left in the box: {leftover_slices}")  # 4

# --- 7. Exponent / Power (**) ---
# 12x12 ইঞ্চির box এর ক্ষেত্রফল (12 এর বর্গ)
box_side = 12
box_area = box_side ** 2
print(f"Area of the pizza box base: {box_area} square inches")  # 144
```

| Operator | কাজ | উদাহরণ | ফলাফল |
|----------|-----|--------|-------|
| `*` | গুণ | `3 * 15` | `45` |
| `+` | যোগ | `45 + 4` | `49` |
| `-` | বিয়োগ | `49 - 5` | `44` |
| `/` | ভাগ (float দেয়) | `44 / 4` | `11.0` |
| `//` | floor division (গোটা অংশ) | `24 // 5` | `4` |
| `%` | modulo (ভাগশেষ) | `24 % 5` | `4` |
| `**` | ঘাত/power | `12 ** 2` | `144` |

---

## ⚖️ ৬. Comparison Operators (তুলনার অপারেটর)

এগুলো দুইটা value তুলনা করে **`True` বা `False`** ফেরত দেয়।

```python
# Setup our base variables
movie_ticket_price = 12
minimum_age = 18
customer_age = 20
customer_cash = 12

print(customer_age > minimum_age)          # True  (>  বড়)
print(movie_ticket_price < customer_cash)  # False (<  ছোট)
print(customer_cash == movie_ticket_price) # True  (== সমান কিনা)
print(customer_age != minimum_age)         # True  (!= সমান নয় কিনা)
print(customer_cash >= movie_ticket_price) # True  (>= বড় বা সমান)
print(customer_age <= 17)                  # False (<= ছোট বা সমান)
```

> ⚠️ **খুব গুরুত্বপূর্ণ:** তুলনা করতে **দুইটা** equal sign `==` লাগে। একটা `=` দিয়ে variable এ value **assign** করা হয়, তুলনা নয়।

---

## 🔗 ৭. Logical Operators (`and`, `or`)

একাধিক শর্ত একসাথে চেক করার জন্য।

```python
# --- and: দুই দিকের শর্তই True হতে হবে ---
not_married = True
age = 20

if not_married == True and age >= 18:
    print("Most eligible bachelor getting married")
else:
    print("Papa will....")
```

```python
# --- or: অন্তত একটা শর্ত True হলেই যথেষ্ট ---
# সব শর্ত fail করলে তবেই False হবে
has_job = True
age = 20

if has_job == True or age >= 18:
    print("Most eligible bachelor getting married")
else:
    print("Okormar dheki, bia korte asche, maira...")
```

| Operator | কখন True হয় |
|----------|-------------|
| `and` | **সব** শর্ত True হলে |
| `or` | **অন্তত একটা** শর্ত True হলে |

---

## 🚦 ৮. Conditions (if / elif / else)

**Condition** এর মূলে আছে **Boolean (`bool`)** data type — যেটার মান শুধু `True` অথবা `False` হতে পারে। এই Boolean দিয়েই আমরা program এর গতিপথ নিয়ন্ত্রণ করি।

Boolean সাধারণত দুইভাবে তৈরি হয়:
1. **সরাসরি assign করে** — `is_active = True`
2. **Comparison operator দিয়ে** — `age >= 18` (এটা `True`/`False` দেয়)

### উদাহরণ: Survey Eligibility

```python
# Raw Data
age = 34
has_kids = True

# 1. শর্ত যাচাই করে Boolean তৈরি
is_adult = age >= 18                          # True
qualifies_for_credit = (age > 30) and has_kids  # True

# 2. Boolean দিয়ে program এর flow নিয়ন্ত্রণ
if is_adult:
    print("Survey access granted.")
else:
    print("Access denied: Must be 18 or older.")
```

### উদাহরণ: Movie Ticket Pricing (if / elif / else)

```python
# এই number বদলে বিভিন্ন path টেস্ট করা যায়!
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

> 🔑 **কীভাবে কাজ করে:** Python উপর থেকে নিচে চেক করে। প্রথম যে শর্ত `True` হয়, শুধু সেটার block চলে, বাকিগুলো skip হয়। উপরে `age = 25` হওয়ায় → "Adult ticket price: $15.00" print হবে।

---

## 🛠️ ৯. Functions (ফাংশন)

Function হলো একটা কাজের logic একবার লিখে বারবার ব্যবহার করার উপায়। `def` keyword দিয়ে তৈরি করা হয়।

```python
def insert_user_in_database(name, age):
    print(f"User inserted in DB for {name}, {age}")

# একটা function কে অবশ্যই call করতে হয়
insert_user_in_database("Alice", 34)
```

### `return` — value ফেরত দেওয়া

`print()` শুধু দেখায়, কিন্তু `return` দিয়ে value ফেরত পাওয়া যায় যা পরে ব্যবহার করা যায়।

```python
def insert_user_in_database(name, age):
    return f"User inserted in DB for {name}, {age}"

print(insert_user_in_database("Alice", 34))
```

### কেন Function দরকার? (বারবার একই কোড না লিখতে)

**❌ Function ছাড়া (একই logic তিনবার লিখতে হচ্ছে):**

```python
# --- Customer 1 (Texas) ---
amount1 = 1000
state1 = "TX"
if state1 == "TX":
    tax_rate1 = 0.0625
elif state1 == "CA":
    tax_rate1 = 0.0725
else:
    tax_rate1 = 0.05
total_tax1 = amount1 * tax_rate1
print(f"Customer 1 Tax: ${total_tax1}")

# ... Customer 2, Customer 3 এর জন্যও একই কোড বারবার লিখতে হয় ...
```

**✅ Function দিয়ে (logic একবার লিখে বারবার reuse):**

```python
# Logic একবারই লেখা হলো
def calculate_tax(amount, state):
    if state == "TX":
        tax_rate = 0.0625
    elif state == "CA":
        tax_rate = 0.0725
    else:
        tax_rate = 0.05
    return amount * tax_rate

# এক লাইনে যতবার খুশি ব্যবহার করা যায়
print(f"Customer 1 Tax: ${calculate_tax(1000, 'TX')}")
print(f"Customer 2 Tax: ${calculate_tax(2500, 'CA')}")
print(f"Customer 3 Tax: ${calculate_tax(500, 'NY')}")
```

> 💡 এটাই **DRY principle** — *Don't Repeat Yourself*। একই কোড বারবার না লিখে function বানিয়ে reuse করো।

---

## 📝 আজকের ক্লাসের সারাংশ

- **Dictionary** = `{key: value}` জোড়া। Key immutable হতে হয় (শুধু string নয়, সংখ্যাও চলে), value যেকোনো type।
- **List** এর কাজ: `append()`, `insert()`, `remove()`, `pop()`, `len()`, `sort()`, `in`।
- **Loop** দিয়ে বারবার কাজ করা যায়; **List Comprehension** সেটাকে এক লাইনে লেখে।
- **Arithmetic:** `+ - * / // % **`
- **Comparison:** `> < == != >= <=` → `True`/`False` দেয়।
- **Logical:** `and` (সব True লাগবে), `or` (একটা True হলেই চলবে)।
- **Conditions:** `if / elif / else` দিয়ে program এর flow নিয়ন্ত্রণ।
- **Functions:** `def` দিয়ে logic একবার লিখে বারবার reuse; `return` দিয়ে value ফেরত।

---

*Happy Coding! 🚀*