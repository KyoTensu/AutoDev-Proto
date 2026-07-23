Worst Errors in Rust Programming
=================================

A summary of some of the most serious and common mistakes made when programming in Rust, and how to avoid them.

1. Fighting the Borrow Checker
------------------------------
Winding around ownership with workarounds (unnecessary clones, interior mutability everywhere, indexed access instead of iterators) produces awkward design instead of embracing ownership. Prefer restructuring data and APIs so lifetimes and exclusive access are obvious.

    // Wrong: fight the checker with clones and shared mutability by default
    fn total(items: &Vec<i32>) -> i32 {
        items.clone().into_iter().sum()
    }

    // Right: take what you need; borrow immutably, iterate by reference
    fn total(items: &[i32]) -> i32 {
        items.iter().sum()
    }

2. Overusing `.clone()` and `.unwrap()`
---------------------------------------
`.clone()` often hides ownership problems instead of fixing them. `.unwrap()` / `.expect()` turn recoverable failures into panics. Prefer borrowing, moving once, and propagating `Result`/`Option` with `?`.

    // Wrong
    let name = user.name.clone();
    let file = File::open(path).unwrap();

    // Right
    let name = &user.name;
    let file = File::open(path)?;

3. Misusing `unsafe`
--------------------
`unsafe` disables some of the compiler's guarantees. Unnecessary blocks increase risk; unsound ones (invalid provenance, missing invariants, data races) are security bugs. Keep `unsafe` minimal, documented, and behind a safe API with clear invariants.

    // Wrong: reach for unsafe to "just" transmute without justifying soundness
    let s: &str = unsafe { std::mem::transmute(bytes) };

    // Right: use safe conversions; isolate irreducible unsafe behind checked APIs
    let s = std::str::from_utf8(bytes)?;

4. Ignoring Lifetimes (Copying to Avoid Them)
---------------------------------------------
Sprinkling `.to_owned()` / `.clone()` everywhere to dodge lifetime annotations bloats allocations and still leaves APIs unclear. Learn to express borrows with lifetime elision and explicit parameters when needed.

    // Wrong: always allocate to avoid thinking about borrows
    fn first_word(s: &str) -> String {
        s.split_whitespace().next().unwrap_or("").to_string()
    }

    // Right: return a borrow tied to the input
    fn first_word(s: &str) -> &str {
        s.split_whitespace().next().unwrap_or("")
    }

5. Overusing `Rc<RefCell<T>>` / `Arc<Mutex<T>>`
-----------------------------------------------
Wrapping everything in shared interior mutability recreates shared-mutable-state spaghetti. Prefer ownership graphs, message passing, or splitting borrows. Use `Rc`/`Arc` and `RefCell`/`Mutex` only where sharing is truly required.

    // Wrong: default architecture is a web of RefCells
    type Shared = Rc<RefCell<Node>>;

    // Right: own data; share only at boundaries that need it
    struct Tree {
        root: Node, // exclusive ownership; children owned in Vec<Node>
    }

6. Blocking the Async Runtime
-----------------------------
Calling blocking I/O, `std::thread::sleep`, or heavy CPU work directly inside `async` tasks stalls the executor and can deadlock other tasks.

    // Wrong
    async fn fetch(url: &str) -> String {
        std::fs::read_to_string("cache.txt").unwrap() // blocking on async worker
    }

    // Right: use async I/O, or move blocking work off the runtime
    async fn fetch(path: &str) -> std::io::Result<String> {
        tokio::fs::read_to_string(path).await
    }
    // or: tokio::task::spawn_blocking(|| std::fs::read_to_string(path)).await?

7. Poor Error Handling
----------------------
Ignoring `Result`, matching only the happy path, or using stringly-typed errors loses context. Leverage `Result`/`Option`, the `?` operator, and structured error types (`thiserror` / `anyhow` as appropriate).

    // Wrong
    let n: i32 = s.parse().unwrap_or(0);
    let _ = do_work(); // discard Result

    // Right
    let n: i32 = s.parse().map_err(|e| AppError::Parse { value: s, source: e })?;
    do_work()?;

8. Holding Locks Across `.await` Points
---------------------------------------
A `std::sync::Mutex` (or any non-async lock) held across an `.await` can block the runtime or deadlock. Drop the guard before awaiting, shrink the critical section, or use an async-aware mutex when the lock must span awaits.

    // Wrong
    async fn bump(state: &Mutex<State>) {
        let mut g = state.lock().unwrap();
        g.n += 1;
        do_async_work().await; // lock held across await
    }

    // Right
    async fn bump(state: &Mutex<State>) {
        {
            let mut g = state.lock().unwrap();
            g.n += 1;
        } // guard dropped
        do_async_work().await;
    }

9. Indexing When Iterators Suffice
----------------------------------
Manual indexing invites off-by-one panics and fights borrow rules. Prefer iterators, slices, and pattern matching.

    // Wrong
    for i in 0..v.len() {
        process(&v[i]);
    }

    // Right
    for item in &v {
        process(item);
    }

10. Confusing `&String` / `&Vec<T>` with `&str` / `&[T]`
--------------------------------------------------------
APIs that take `&String` or `&Vec<T>` force callers to own heap data. Accept `&str` and `&[T]` (or `impl AsRef<str>`) so both owned and borrowed values work.

    // Wrong
    fn len(s: &String) -> usize { s.len() }

    // Right
    fn len(s: &str) -> usize { s.len() }

11. Not Implementing `Send` / `Sync` Consciously (and Sending Non-thread-safe Types)
------------------------------------------------------------------------------------
Moving a type that is not `Send` into another thread (or holding `!Sync` data across threads incorrectly) fails at compile time — or, with `unsafe`, becomes a data race. Design types for their concurrency needs; don't bypass checks.

    // Wrong idea: wrap non-thread-safe interior in Arc and share across threads without Sync
    // Right: use Arc<Mutex<T>> / Arc<RwLock<T>> for shared mutation, or message passing

12. Infinite-size Recursive Types Without Indirection
-----------------------------------------------------
A struct that contains itself by value has infinite size and will not compile. Box (or another indirection) recursive fields.

    // Wrong
    struct Node { child: Node }

    // Right
    struct Node { child: Option<Box<Node>> }

13. Forgetting to Handle Poisoned Mutexes or Cancellation
---------------------------------------------------------
`Mutex::lock()` returns `LockResult`; blindly `.unwrap()` panics if another thread poisoned the lock. In async code, similarly treat cancellation and join errors as real outcomes, not ignore them.

    // Wrong
    let guard = mutex.lock().unwrap();

    // Right
    let guard = mutex.lock().unwrap_or_else(|p| p.into_inner());
    // or propagate / recover explicitly based on program policy

14. Premature or Misplaced `collect`
------------------------------------
Collecting an iterator just to iterate again wastes allocation and can break short-circuiting (e.g. with `?` or `find`). Keep chains lazy until a concrete collection is required.

    // Wrong
    let items: Vec<_> = iter.map(transform).collect();
    for x in items { use_item(x); }

    // Right
    for x in iter.map(transform) { use_item(x); }

15. Resource Cleanup Only via `drop` Logic Callers Forget
---------------------------------------------------------
Relying on callers to call a custom `close()` is error-prone. Implement `Drop` for necessary cleanup, and prefer RAII guards so resources release on all paths (including panic unwinding where safe).

    // Wrong: easy to forget
    file.close();

    // Right: File (and most std I/O types) already drop/cleanup; use scopes and ownership
    {
        let mut f = File::create(path)?;
        write_all(&mut f)?;
    } // closed here

---

Avoiding these pitfalls leads to code that is more idiomatic, safer, and maintainable.
