# Lab 1 Notes — Python Basics: BVRIT Canteen Billing System

All 10 concepts below build on **one running example** — a canteen billing system. It starts as simple as billing a single samosa, then grows step by step into a small system with a menu, a cart, a reusable billing function, saved receipts, and error handling. Notice *why* each new concept gets introduced — every one solves a limitation of the version before it.

Try running and modifying the examples yourself in **Lab1_Practice_Exercises.ipynb**.

---

## 1. Variables & Data Types

A variable is a labelled box that stores a value. Python figures out the data type automatically based on what you store.

**Canteen example:** Recording a single item order at the counter.

```python
item_name = "Samosa"     # str   -> text
price = 15.0              # float -> decimal number (money)
quantity = 2               # int   -> whole number
is_veg = True               # bool  -> True/False

print(item_name, price, quantity, is_veg)
print(type(item_name), type(price), type(quantity), type(is_veg))
```

This works fine for **one item**. Keep this limitation in mind — it comes back in Section 4.

---

## 2. Operators — Billing One Item

Arithmetic operators (`+ - * / % **`) let you compute values from your variables.

**Canteen example:** Turning that one order into a bill.

```python
subtotal = price * quantity
gst_rate = 0.05                     # 5% GST
gst_amount = subtotal * gst_rate
total = subtotal + gst_amount

print(f"Subtotal: Rs.{subtotal}")
print(f"GST (5%): Rs.{gst_amount}")
print(f"Total: Rs.{total}")
```

---

## 3. Conditional Statements — Loyalty Discount & Parcel Charge

`if / elif / else` lets a program make decisions instead of always doing the same calculation.

**Canteen example:** The canteen gives a loyalty discount on larger bills, and adds a packing charge for parcel (takeaway) orders.

```python
is_parcel = True

if total > 200:
    discount_rate = 0.10
elif total > 100:
    discount_rate = 0.05
else:
    discount_rate = 0.0

discount_amount = total * discount_rate
packing_charge = 5 if is_parcel else 0

final_total = total - discount_amount + packing_charge

print(f"Discount ({discount_rate*100:.0f}%): -Rs.{discount_amount}")
print(f"Packing charge: Rs.{packing_charge}")
print(f"Final total: Rs.{final_total}")
```

---

## 4. Loops — Billing a Full Order (Multiple Items)

Loops repeat an action without writing it out manually every time — needed the moment an order has more than one item.

**Canteen example:** A student orders several items in one go. Since we only know variables and lists of raw values so far, we track item names and prices in two **parallel lists** — notice how easy it is for these to get out of sync (item 3's name might not line up with item 3's price if you're not careful). This limitation motivates Section 7.

```python
order_items = ["Samosa", "Tea", "Vada"]
order_prices = [15.0, 10.0, 12.0]
order_qty    = [2, 1, 3]

order_subtotal = 0
for i in range(len(order_items)):
    line_total = order_prices[i] * order_qty[i]
    order_subtotal += line_total
    print(f"{order_items[i]} x{order_qty[i]} = Rs.{line_total}")

print(f"Order subtotal: Rs.{order_subtotal}")
```

---

## 5. Strings — Formatting the Receipt Header

Strings represent text, and Python has built-in tools to reshape them.

**Canteen example:** A customer's name is typed messily at the counter; clean it up before printing the receipt.

```python
raw_name = "  ananya RAO  "

clean_name = raw_name.strip().title()     # remove extra spaces, fix capitalisation
receipt_header = f"Receipt for: {clean_name}"

print(f"Original : '{raw_name}'")
print(receipt_header)
```

---

## 6. Lists — Building an Order Cart

A list stores an ordered, changeable collection of items — perfect for a cart a customer builds item by item.

**Canteen example:** Adding and removing items as the customer changes their mind at the counter.

```python
cart = ["Samosa", "Tea"]

cart.append("Vada")             # add an item
cart.remove("Tea")              # customer changed their mind
cart.insert(1, "Cold Coffee")   # insert at a specific position

print("Current cart:", cart)
print("Number of items:", len(cart))
print("Is 'Samosa' in the cart?", "Samosa" in cart)
```

This tells us **what's in the cart**, but not the price of each item — for that we still need the menu's parallel price list from Section 4. That's exactly the gap a dictionary closes.

---

## 7. Dictionaries — The Menu (Fixing the Parallel-List Problem)

A dictionary stores data as `key: value` pairs. Instead of keeping item names and prices in two separate lists that can drift out of sync, one dictionary holds both together.

**Canteen example:** The full canteen menu, and totalling the cart from Section 6 against it.

```python
menu = {
    "Samosa": 15.0,
    "Tea": 10.0,
    "Vada": 12.0,
    "Cold Coffee": 30.0
}

cart = ["Samosa", "Cold Coffee", "Vada"]

order_subtotal = 0
for item in cart:
    price = menu[item]           # direct lookup - no risk of index mismatch
    order_subtotal += price
    print(f"{item}: Rs.{price}")

print(f"Order subtotal: Rs.{order_subtotal}")
```

Compare this to Section 4 — no more keeping two lists lined up by position. Add a new menu item, and it's just one new key-value pair.

---

## 8. Functions — A Reusable `calculate_bill()`

A function packages reusable logic so the discount, GST, and packing-charge rules from Sections 2–3 don't have to be retyped for every order.

**Canteen example:** One function that bills *any* cart, for *any* customer.

```python
def calculate_bill(cart, menu, is_parcel=False, gst_rate=0.05):
    subtotal = sum(menu[item] for item in cart)

    if subtotal > 200:
        discount_rate = 0.10
    elif subtotal > 100:
        discount_rate = 0.05
    else:
        discount_rate = 0.0

    discount_amount = subtotal * discount_rate
    gst_amount = (subtotal - discount_amount) * gst_rate
    packing_charge = 5 if is_parcel else 0
    total = subtotal - discount_amount + gst_amount + packing_charge

    return {
        "subtotal": subtotal,
        "discount": discount_amount,
        "gst": gst_amount,
        "packing_charge": packing_charge,
        "total": round(total, 2)
    }

bill = calculate_bill(["Samosa", "Cold Coffee", "Vada"], menu, is_parcel=True)
print(bill)
```

Now billing a second, completely different order takes one line — no copy-pasting the discount logic again:

```python
bill2 = calculate_bill(["Tea", "Tea"], menu, is_parcel=False)
print(bill2)
```

---

## 9. File Handling — Saving the Day's Receipts

Programs often need to save data permanently instead of losing it when the program ends.

**Canteen example:** Appending each billed order to a daily sales log, then reading the full day's log back.

```python
def save_receipt(customer_name, bill):
    with open("canteen_sales_log.txt", "a") as f:
        f.write(f"{customer_name}: Rs.{bill['total']}\n")

save_receipt("Ananya Rao", bill)
save_receipt("Ravi Kumar", bill2)

# Read the full day's log back
with open("canteen_sales_log.txt", "r") as f:
    print(f.read())
```

Note the `"a"` (append) mode — each new order is added without erasing the previous ones.

---

## 10. Exception Handling — A Safer Billing Counter

`try / except` lets a program handle errors gracefully instead of crashing — essential once real (imperfect) input is involved at the counter.

**Canteen example:** Handling an item that isn't on the menu, and a customer paying less cash than the bill total.

```python
def bill_and_collect_cash(cart, menu, cash_paid, is_parcel=False):
    try:
        bill = calculate_bill(cart, menu, is_parcel)
        if cash_paid < bill["total"]:
            raise ValueError(f"Insufficient cash: paid Rs.{cash_paid}, bill is Rs.{bill['total']}")
        change = round(cash_paid - bill["total"], 2)
        print(f"Bill: Rs.{bill['total']} | Change: Rs.{change}")
        return bill
    except KeyError as e:
        print(f"Item not on menu: {e}")
    except ValueError as e:
        print(f"Transaction failed: {e}")

bill_and_collect_cash(["Samosa", "Dosa"], menu, cash_paid=50)   # "Dosa" isn't on the menu -> KeyError
bill_and_collect_cash(["Samosa", "Tea"], menu, cash_paid=10)     # not enough cash -> ValueError
bill_and_collect_cash(["Samosa", "Tea"], menu, cash_paid=100)    # succeeds
```

---

## Quick Reference — How the Canteen System Grew

| # | Concept | What it added to the canteen system |
|---|---|---|
| 1 | Variables & data types | Store one order's details |
| 2 | Operators | Compute subtotal, GST, total |
| 3 | Conditionals | Discount tiers + parcel charge |
| 4 | Loops | Bill an order with multiple items |
| 5 | Strings | Clean up the receipt header |
| 6 | Lists | A changeable order cart |
| 7 | Dictionaries | A proper menu (fixes parallel-list drift) |
| 8 | Functions | One reusable `calculate_bill()` for any order |
| 9 | File handling | Persist each day's receipts to disk |
| 10 | Exceptions | Handle bad items / insufficient cash safely |

Practice exercises for this material are in **Lab1_Practice_Exercises.ipynb**.
