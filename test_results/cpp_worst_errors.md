Worst Errors in C/C++ Programming
=================================

A summary of some of the most serious and common mistakes made when programming in C and C++, and how to avoid them.

1. Buffer Overflows
-------------------
Writing past the end of an array corrupts memory, crashes the program, or opens a security hole. Always bound your writes.

    char buf[8];                         // Wrong
    strcpy(buf, "this string is too long");

    char buf[8];                         // Right
    snprintf(buf, sizeof(buf), "%s", src);

2. Using Memory After free / delete (Dangling Pointers)
-------------------------------------------------------
Accessing memory after it has been released is undefined behavior. Set pointers to null after freeing.

    free(p);                             // Wrong
    printf("%d\n", *p);

    free(p);                             // Right
    p = NULL;

3. Memory Leaks
---------------
Every allocation must have a matching deallocation. In C++, prefer smart pointers and RAII over manual new/delete.

    int *p = new int[100];               // Wrong (never deleted)

    auto p = std::make_unique<int[]>(100);   // Right (freed automatically)

4. Uninitialized Variables
--------------------------
Reading a variable before assigning it yields an indeterminate value and non-deterministic bugs.

    int count;                           // Wrong
    total += count;

    int count = 0;                       // Right
    total += count;

5. Off-by-One Errors
--------------------
Iterating one element too far reads or writes out of bounds. Valid indices for size n run from 0 to n-1.

    for (int i = 0; i <= n; i++)         // Wrong
        arr[i] = 0;

    for (int i = 0; i < n; i++)          // Right
        arr[i] = 0;

6. Confusing "=" with "==" in Conditions
----------------------------------------
Assignment inside a condition compiles but is almost never intended.

    if (x = 5) do_work();                // Wrong (assigns, always true)

    if (x == 5) do_work();               // Right

7. Forgetting the Null Terminator on C Strings
----------------------------------------------
C strings must end in a NUL byte. Missing it makes string functions run off into adjacent memory.

    char s[3];                           // Wrong (no room for the NUL)
    memcpy(s, "abc", 3);
    printf("%s\n", s);

    char s[] = "abc";                    // Right (terminator added)

8. Mismatched Allocation and Deallocation
-----------------------------------------
Mixing allocation families is undefined behavior. Pair malloc/free, new/delete, and new[]/delete[].

    int *p = new int[10];                // Wrong
    delete p;

    int *p = new int[10];                // Right
    delete[] p;

9. Integer Overflow and Signed/Unsigned Mismatch
------------------------------------------------
Signed overflow is undefined behavior, and comparing signed with unsigned produces surprising results.

    unsigned int u = 1;                  // Wrong
    int i = -1;
    if (i < u) ...            // false: i converts to a huge unsigned value

    if (i < 0 || (unsigned)i < u) ...    // Right

10. Returning Pointers or References to Local Variables
-------------------------------------------------------
A local variable's lifetime ends when the function returns; using its address afterward is undefined.

    int* make() {                        // Wrong
        int x = 42;
        return &x;
    }

    int make() { return 42; }            // Right (return by value)

11. Missing break in switch Statements
--------------------------------------
Without break, execution falls through to the next case, often unintentionally.

    switch (n) {                         // Wrong
        case 1: do_one();
        case 2: do_two();
    }

    switch (n) {                         // Right
        case 1: do_one(); break;
        case 2: do_two(); break;
    }

12. Undefined Behavior from Multiple Modifications in One Expression
-------------------------------------------------------------------
Modifying the same variable more than once without a sequence point is undefined.

    i = i++ + ++i;                       // Wrong (undefined)

    i++;                                 // Right (split into statements)
    ++i;

13. Not Checking Return Values
------------------------------
Functions like malloc, fopen, and read can fail. Ignoring their return values causes crashes on the next use.

    FILE *f = fopen("data.txt", "r");    // Wrong
    fread(buf, 1, n, f);

    FILE *f = fopen("data.txt", "r");    // Right
    if (f == NULL) return;

14. Violating the Rule of Three / Five (C++)
--------------------------------------------
A class that manages a resource and defines one of destructor, copy constructor, or copy assignment usually needs all of them (and the move operations in modern C++), or resources get double-freed or leaked. Prefer RAII types that need none of them.

15. Dereferencing Null Pointers
-------------------------------
Following a null (or otherwise invalid) pointer is undefined behavior and usually crashes. Check before you dereference.

    int *p = find(key);                  // Wrong
    return *p;

    int *p = find(key);                  // Right
    if (p == nullptr) return -1;
    return *p;

16. Comparing Floating-Point Numbers with ==
--------------------------------------------
Rounding makes exact equality unreliable. Compare within a tolerance instead.

    if (a == b) ...                      // Wrong

    if (std::fabs(a - b) < 1e-9) ...     // Right

17. Mismatched printf / scanf Format Specifiers
-----------------------------------------------
The format string must match the argument types, or you get garbage output or memory corruption.

    long n = 5;                          // Wrong
    printf("%d\n", n);
    scanf("%d", &n);

    long n = 5;                          // Right
    printf("%ld\n", n);
    scanf("%ld", &n);

18. Using sizeof on a Pointer Instead of an Array
-------------------------------------------------
Arrays decay to pointers when passed to functions, so sizeof then measures the pointer, not the array.

    void f(int arr[]) {                  // Wrong
        int n = sizeof(arr) / sizeof(arr[0]);   // always sizeof(pointer)
    }

    void f(int arr[], size_t n) { ... }  // Right (pass the length explicitly)

19. Unparenthesized Macro Arguments
-----------------------------------
Text substitution ignores operator precedence, so macro arguments and bodies need parentheses.

    #define SQ(x) x * x                  // Wrong
    int y = SQ(a + b);                   // expands to a + b * a + b

    #define SQ(x) ((x) * (x))            // Right
    int y = SQ(a + b);                   // or better: a constexpr function

20. Modifying a Container While Iterating (C++)
-----------------------------------------------
Inserting or erasing invalidates iterators; using a stale iterator is undefined. Use the value erase returns.

    for (auto it = v.begin(); it != v.end(); ++it)   // Wrong
        if (*it == 0) v.erase(it);

    for (auto it = v.begin(); it != v.end(); )       // Right
        it = (*it == 0) ? v.erase(it) : it + 1;

---

Avoiding these pitfalls leads to code that is more correct, safer, and maintainable.
