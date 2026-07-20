Worst Errors in Python Programming
==================================

A summary of some of the most serious and common mistakes made when programming in Python, and how to avoid them.

1. Mutable Default Arguments
----------------------------
Using a mutable object (like a list or dict) as a default argument value. The default is created once at function definition time and shared across all calls.

    def add_item(item, items=[]):        # Wrong
        items.append(item)
        return items

    def add_item(item, items=None):      # Right
        if items is None:
            items = []
        items.append(item)
        return items

2. Bare except Clauses
----------------------
Catching every exception hides bugs and swallows signals like KeyboardInterrupt and SystemExit.

    try:                                 # Wrong
        do_work()
    except:
        pass

    try:                                 # Right
        do_work()
    except ValueError as exc:
        logging.error("Invalid value: %s", exc)

3. Modifying a Collection While Iterating Over It
-------------------------------------------------
Adding to or removing from a list or dict during iteration causes skipped elements or a RuntimeError.

    for item in items:                   # Wrong
        if should_remove(item):
            items.remove(item)

    items = [x for x in items if not should_remove(x)]   # Right

4. Confusing "is" with "=="
---------------------------
The "is" operator checks object identity, while "==" checks value equality. Using "is" for value comparison works by accident for small ints and interned strings, then breaks silently.

    if value is 1000:      # Wrong
        ...

    if value == 1000:      # Right
        ...

5. Integer / Float and Division Surprises
-----------------------------------------
In Python 3, "/" always returns a float and "//" does floor division. Mixing them up produces wrong results.

    7 / 2      -> 3.5
    7 // 2     -> 3
    -7 // 2    -> -4   (floors toward negative infinity)

6. Shadowing Built-in Names
---------------------------
Naming a variable list, dict, str, id, type, or sum overrides the built-in for the rest of the scope.

    list = [1, 2, 3]       # Wrong
    new = list((4, 5))     # TypeError: list object is not callable

7. Floating-Point Equality Comparisons
---------------------------------------
Floats are approximate, so direct equality checks fail unexpectedly.

    0.1 + 0.2 == 0.3               # False

    import math                    # Right
    math.isclose(0.1 + 0.2, 0.3)   # True

8. Late Binding in Closures / Loops
-----------------------------------
Closures capture variables, not values. All closures created in a loop see the final value.

    funcs = [lambda: i for i in range(3)]     # Wrong
    [f() for f in funcs]        -> [2, 2, 2]

    funcs = [lambda i=i: i for i in range(3)] # Right
    [f() for f in funcs]        -> [0, 1, 2]

9. Not Closing Files or Resources
---------------------------------
Failing to close files can leak file descriptors and lose unflushed writes. Always use context managers.

    f = open("data.txt")   # Wrong
    data = f.read()

    with open("data.txt") as f:   # Right
        data = f.read()

10. Mixing Tabs and Spaces for Indentation
-------------------------------------------
Python treats indentation as syntax. Mixing tabs and spaces raises a TabError or, worse, silently changes control flow. Use 4 spaces consistently (PEP 8).

11. Circular Imports
--------------------
Two modules importing each other at module level cause an ImportError or partially initialized modules. Refactor shared code into a third module or move the import inside a function.

12. Ignoring Exceptions from Type Coercion
-------------------------------------------
Blindly converting user input without handling failure crashes the program.

    age = int(user_input)  # Wrong

    try:                   # Right
        age = int(user_input)
    except ValueError:
        age = None

---

Avoiding these pitfalls leads to code that is more correct, readable, and maintainable.
