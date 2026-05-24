# Learning Log

A record of concepts and lessons learned while building this project.

---

## Architecture: capturing state in a struct

**Question:** The search algorithm needs the target coordinates for comparison, but the board is `[][]int` which means scanning the whole grid to find it. Should a struct hold this?

**Answer:** Yes. The board and its answer are logically one thing. A struct keeps them together and avoids re-scanning on every guess:

```go
type Board struct {
    grid    [][]int
    targetX int
    targetY int
}
```

`setUpBoard` returns `*Board` instead of `[][]int`, storing the target coordinates at construction time. The search function can then accept the actual answer directly instead of searching for it.

---

## panic: invalid argument to IntN

`rand.IntN` panics if passed a value of `0` or less. This happens when a search window collapses — `xmax - xmin` becomes zero or negative. The root cause is usually incorrect window-narrowing logic that grows the window instead of shrinking it, or shrinks it past the target.

---

## Window narrowing: which bound to move

When the signed delta (`diff = target - guess`) comes back from a missed guess:

- `diff > 0` means the target is **above/right** of the guess → raise the **lower** bound: `xmin = xguess + 1`
- `diff < 0` means the target is **below/left** of the guess → lower the **upper** bound: `xmax = xguess`
- `diff == 0` means the guess is already on the correct coordinate in that axis → **don't adjust** that bound at all

Getting these backwards causes the window to expand rather than contract, eventually producing negative window sizes and a panic.

---

## Exclusive bounds and the +1 rule

When `diff > 0`, setting `xmin = xguess` (without `+1`) means the new lower bound still includes the current guess — a coordinate already known to be wrong. Using `xmin = xguess + 1` makes the bound exclusive, ensuring the search never wastes a guess on a coordinate already ruled out.

Both axes must use the same convention (both exclusive) or the window can get stuck. An asymmetric implementation — where one axis uses exclusive bounds and the other doesn't — can cause infinite loops.

---

## rand.IntN range and shifting

`rand.IntN(n)` returns a value in `[0, n)`. To get a random value within a window `[xmin, xmax)`:

```go
xguess = rand.IntN(xmax-xmin) + xmin
```

Without `+ xmin`, you always guess from the origin regardless of where the window is.

---

## Random search vs midpoint search

Both use the same window-narrowing logic. The difference is how the next guess is chosen:

- **Random:** `rand.IntN(xmax-xmin) + xmin` — unpredictable window reduction, high variance
- **Midpoint:** `(xmin + xmax) / 2` — always halves the window, guaranteed `O(log n)` convergence

On an 800×800 board:

| Method   | Worst case       | Average       | Variance |
|----------|-----------------|---------------|----------|
| Random   | up to 800 guesses | ~log2(800)   | High     |
| Midpoint | ~20 guesses      | ~20 guesses  | None     |

The averages are similar, but midpoint wins on **consistency** — it has no unlucky runs. Integer division handles the midpoint naturally in Go with no rounding needed.

---

## Efficiency comparison formula

```
efficiency = (randomGuesses - midpointGuesses) / randomGuesses * 100
```

This expresses how many fewer guesses midpoint needed as a percentage of random's total. Watch out for `NaN`: if `randomGuesses == 0` (the first guess was correct), this divides by zero. Go's float arithmetic returns `NaN` rather than panicking, so it must be guarded explicitly:

```go
if randomGuesses == 0 {
    return 100.0
}
```

---

## Average of averages

Running a single game gives a noisy result due to random variance. Running many games and averaging the efficiency percentages gives a more reliable picture. This is called an average of averages — each game produces one efficiency value, and those are averaged together.

---

## Concurrency with goroutines and channels

Each game is independent (its own board, its own random target), making the workload trivially parallelisable. The Go pattern for this:

```go
results := make(chan float64, games)  // buffered channel

for range games {
    go func() {
        results <- runGame()          // each goroutine sends one result
    }()
}

total := 0.0
for range games {
    total += <-results                // main collects all results
}
```

Key points:
- `go func()` spawns a goroutine — a lightweight concurrent function
- `chan float64` is a channel that carries float values between goroutines
- The buffer size (`games`) lets all goroutines send without blocking, even before main starts reading
- Main reads from the channel exactly as many times as there are goroutines, then averages

Without the buffer, goroutines would block waiting for main to receive each value one at a time, removing the concurrency benefit.

---

## Idiomatic Go style

Things that don't match idiomatic Go and why:

| Non-idiomatic | Idiomatic | Reason |
|---|---|---|
| `if x { return } else { ... }` | drop the `else` | the `else` is unreachable after a `return` |
| `guesses += 1` | `guesses++` | Go has a `++` statement |
| `setUpBoard()` | `newBoard()` | Go constructors are named `New<Type>` or `new<Type>` |
| `== 1` / `= 1` literals | named constant `ship = 1` | magic numbers are hard to read and change |
| commented-out debug prints | delete them | committed code should be clean |

---

## Functions as parameters

Go functions are first-class values — they can be passed as arguments, stored in variables, and returned from other functions. This is useful for removing duplicated logic where only one small part differs between implementations.

In this project, `randomSearch` and `midpointSearch` were identical except for how the next guess was calculated. By extracting that one difference into a `next` function parameter, both strategies share a single `search` function:

```go
// next receives the current window bounds and returns the next guess
func search(board *Board, xguess, yguess int, next func(xmin, xmax, ymin, ymax int) (int, int)) int {
    ...
    xguess, yguess = next(xmin, xmax, ymin, ymax)
}
```

The two strategies become small standalone functions:

```go
func randomNext(xmin, xmax, ymin, ymax int) (int, int) {
    return rand.IntN(xmax-xmin) + xmin, rand.IntN(ymax-ymin) + ymin
}

func midpointNext(xmin, xmax, ymin, ymax int) (int, int) {
    return (xmin + xmax) / 2, (ymin + ymax) / 2
}
```

And the call site passes the function by name, without calling it (no parentheses):

```go
search(board, xguess, yguess, randomNext)
search(board, xguess, yguess, midpointNext)
```

The tradeoff is added complexity — the function signature is harder to read at a glance, and a reader needs to understand the pattern to follow the code. It's worth it when the duplication is significant; for small functions it may not be.

---

## Pass by value in Go

Go passes integers by value, not by reference. A function that takes `min, max int` and modifies them only modifies its own local copies — the caller's variables are unchanged. To propagate changes back, either return the new values (idiomatic) or pass pointers. Multiple return values are the preferred Go approach:

```go
func narrowWindow(...) (int, int, int, int) {
    ...
    return xmin, xmax, ymin, ymax
}
xmin, xmax, ymin, ymax = narrowWindow(...)
```

---

## Memory cost of large grids

Each `Board` allocates a full `width × height` grid. At `width = height = 80000`, that is:

```
80000 × 80000 × 8 bytes = ~51 GB per board
```

Running 100 concurrent games at that size would require ~5 TB of RAM. For experiments with many concurrent games, keep board dimensions small (e.g. 800×800 = ~5 MB per board).
