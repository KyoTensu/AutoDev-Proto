# The Worst Errors in Python Programming

A summary of the most common and damaging mistakes that Python developers
can make, along with why they are harmful and how to avoid them.

## 1. Mutable Default Arguments

Using a mutable object (list, dict, set) as a default argument value.

```python
def add_item(item, items=[]):  # Bug: the list is shared across calls
    items.append(item)
    return items
```

The default list is created once at function definition and reused on every
call, leading to surprising shared state. Use `None` as the sentinel instead:

```python
def add_item(item, items=None):
    if items is None:
        items = []
    items.append(item)
    return items
```

## 2. Bare `except` Clauses

Catching everything hides real bugs and makes debugging nearly impossible.

```python
try:
    risky()
except:  # Swallows KeyboardInterrupt, SystemExit, and all real errors
    pass
```

Catch only the specific exceptions you expect, and never silently `pass`.

## 3. Comparing with `==` instead of `is` for `None`

```python
if value == None:   # Wrong
if value is None:   # Correct
```

`None` is a singleton; identity comparison is both faster and correct.

## 4. Modifying a Collection While Iterating Over It

```python
for item in my_list:
    if condition(item):
        my_list.remove(item)  # Skips elements and causes subtle bugs
```

Iterate over a copy or build a new list with a comprehension instead.

## 5. Confusing `==` (equality) with `=` (assignment)

While Python raises a `SyntaxError` in most conditionals, misusing them in
comprehensions or walrus expressions can produce logic errors.

## 6. Integer/Float Division Confusion

```python
result = 5 / 2    # 2.5 (float division)
result = 5 // 2   # 2   (floor division)
```

Using the wrong operator silently produces incorrect numeric results.

## 7. Shadowing Built-in Names

```python
list = [1, 2, 3]   # Now the built-in list() is broken in this scope
type = "example"   # Shadows the built-in type()
```

Avoid naming variables after built-ins like `list`, `dict`, `str`, `id`, `type`.

## 8. Ignoring Floating-Point Precision

```python
0.1 + 0.2 == 0.3   # False!
```

Use `math.isclose()` or the `decimal` module for exact comparisons.

## 9. Misusing `global` and Mutable State

Overusing global variables makes code hard to test and reason about, and
leads to hidden dependencies between functions.

## 10. Not Closing Resources

Failing to close files, sockets, or database connections leaks resources.

```python
f = open("file.txt")   # Risky: may never be closed on error
# Better:
with open("file.txt") as f:
    data = f.read()
```

The `with` statement guarantees cleanup even if an exception occurs.

## 11. Circular Imports

Two modules importing each other cause `ImportError` or partially
initialized modules. Restructure code or import locally inside functions.

## 12. Blocking the Event Loop in Async Code

Calling synchronous, long-running functions inside `async` code blocks the
entire event loop, defeating the purpose of concurrency.

---

Avoiding these pitfalls leads to more reliable, readable, and maintainable
Python code.
