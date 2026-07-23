Worst Errors in Java Programming
================================

A summary of some of the most serious and common mistakes made when programming in Java, and how to avoid them.

1. Comparing Objects with == Instead of equals()
------------------------------------------------
The "==" operator compares references (identity), not contents. For objects, especially Strings, this works by accident when values are interned and then breaks silently.

    if (a == b) { ... }          // Wrong (compares references)

    if (a.equals(b)) { ... }     // Right (compares values)
    if (Objects.equals(a, b)) {} // Right (null-safe)

2. NullPointerException from Unchecked References
-------------------------------------------------
Dereferencing a null reference is the most common runtime failure in Java. Guard against it or use Optional.

    String name = user.getName();
    int len = name.length();     // Wrong: name may be null

    int len = Optional.ofNullable(user.getName())
                      .map(String::length)
                      .orElse(0); // Right

3. Ignoring or Swallowing Exceptions
------------------------------------
Catching an exception and doing nothing hides bugs and makes failures impossible to diagnose.

    try {                        // Wrong
        doWork();
    } catch (Exception e) {
        // ignored
    }

    try {                        // Right
        doWork();
    } catch (IOException e) {
        logger.error("Work failed", e);
    }

4. Not Closing Resources
------------------------
Streams, connections, and files leak if not closed. Use try-with-resources so they close automatically, even on exceptions.

    InputStream in = new FileInputStream(path); // Wrong
    in.read();

    try (InputStream in = new FileInputStream(path)) { // Right
        in.read();
    }

5. Mutating a Collection While Iterating Over It
------------------------------------------------
Adding to or removing from a collection during a for-each loop throws ConcurrentModificationException.

    for (String s : list) {          // Wrong
        if (s.isEmpty()) list.remove(s);
    }

    list.removeIf(String::isEmpty);  // Right

6. Integer Division and Overflow Surprises
------------------------------------------
Dividing two ints truncates toward zero, and arithmetic silently overflows the int range.

    double avg = sum / count;        // Wrong if both are int
    double avg = (double) sum / count; // Right

    int big = Integer.MAX_VALUE + 1; // Wrong: overflows to negative
    long big = (long) Integer.MAX_VALUE + 1; // Right

7. Using Floating-Point for Money or Exact Values
-------------------------------------------------
double and float are approximate, so equality checks and sums drift.

    0.1 + 0.2 == 0.3            // false

    BigDecimal total = new BigDecimal("0.1")   // Right
                          .add(new BigDecimal("0.2"));

8. Autoboxing Pitfalls and Cached Integer Identity
--------------------------------------------------
Comparing boxed Integers with "==" only works for the cached range (-128 to 127), then fails. Autoboxing a null Integer into an int throws NPE.

    Integer a = 1000, b = 1000;
    if (a == b) { ... }          // Wrong: false for large values
    if (a.equals(b)) { ... }     // Right

    Integer n = map.get(key);
    int x = n;                   // Wrong: NPE if key absent

9. Not Overriding hashCode() When Overriding equals()
-----------------------------------------------------
If equals() is overridden but hashCode() is not, objects break in HashMap and HashSet: equal objects land in different buckets.

    @Override public boolean equals(Object o) { ... } // Wrong: alone
    @Override public int hashCode() {                 // Right: both
        return Objects.hash(field1, field2);
    }

10. Concatenating Strings in a Loop
-----------------------------------
Strings are immutable, so "+" in a loop creates a new object each iteration, giving O(n^2) behavior. Use StringBuilder.

    String out = "";                 // Wrong
    for (String s : parts) out += s;

    StringBuilder sb = new StringBuilder(); // Right
    for (String s : parts) sb.append(s);

11. Raw Types Instead of Generics
---------------------------------
Using raw types defeats compile-time type checking and invites ClassCastException at runtime.

    List list = new ArrayList();     // Wrong
    list.add("text");
    Integer n = (Integer) list.get(0); // ClassCastException

    List<String> list = new ArrayList<>(); // Right

12. Non-Thread-Safe Access to Shared State
------------------------------------------
Reading and writing shared mutable state from multiple threads without synchronization causes race conditions and stale reads.

    private int count;               // Wrong
    public void inc() { count++; }   // not atomic

    private final AtomicInteger count = new AtomicInteger(); // Right
    public void inc() { count.incrementAndGet(); }

---

Avoiding these pitfalls leads to code that is more correct, robust, and maintainable.
