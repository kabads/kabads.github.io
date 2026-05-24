---
layout: post
title: Battleships in Go - learning log
date: 2026-05-24
categories:
- tech
giscus: true
---

# Learning Log

I have been learning more [Go](https://go.dev) over the past week or so. I decided a good project would be a battleship game, but I prefer games where there is quite a bit of automation (this one was wholly automated). Basically, we create a grid, and somewhere in the grid is a single ship (x, y coordinates). The goal is to find that. The first strategy was randomly finding it. This was amazingly inefficient, so the next strategy was to provide min/max concept, whereby after an incorrect search, we provide how far away we are from the correct coordinates.

The next improvement after providing min / max, was to choose a set of random coordinates within this range. This was more efficient, but not the best method. The next iteration was to find the midpoint of the range - this meant that we reduced the range by 50% every time. Another efficiency saving.

Then I decided to run many games. Until this point, I had only been running one battleship game, but comparing the two search strategies (random and midpoint). With random searches, you will get a high variance of results. To reduce the variance, we can run many games, and then get an average of these results.

Below are the learning points that I found during this fun project. I find writing about them consolidates my learning.

The final code for this exercise can be found here [https://github.com/kabads/battleships-go](https://github.com/kabads/battleships-go).

---

## Architecture: capturing state in a struct

Initially I stored the grid as a `[][]int` type. This was a good starting point, but later on, I also wanted to store the solve (coordinates of the ship that we are searching for). I could brute-force the search again to get the difference to update the min / max values. However, that was very inefficient, as I already had that information. The best way to manage this was to create a struct that held that information as well as the grid:

```go
type Board struct {
    grid    [][]int
    targetX int
    targetY int
}
```

This meant that the board is now a struct of values and the search function can accept this.

I had a function called `setUpBoard()` that accepted `[][]int` but this changed to be a pointer to a type `*Board`. This meant the board was passed around to each function, and now with the target coordinates.  

The `setUpBoard()` function was renamed to `newBoard()` to make it more idiomatic. In Go, constructors are typically named `New<Type>` or `new<Type>`, and they return a pointer to the new struct. This is a common pattern in Go for creating and initializing complex types.

---

## panic: invalid argument to IntN

I rarely started to get a panic with `rand.IntN(n)` calls. This is because `xmin - xmax` was giving zero. You cannot pass this in to the random function as it will panic. This was caused by the window narrowing logic being incorrect, and the window expanding rather than contracting. I fixed this by ensuring that when the target is above/right of the guess, I raise the lower bound (`xmin = xguess + 1`), and when the target is below/left of the guess, I lower the upper bound (`xmax = xguess`). This is nothing to do with Go itself, but I wanted to recognise this logic for the future.

---

## Window narrowing: which bound to move

When the signed delta (`diff = target - guess`) comes back from a missed guess:

- `diff > 0` means the target is **above/right** of the guess → raise the **lower** bound: `xmin = xguess + 1`
- `diff < 0` means the target is **below/left** of the guess → lower the **upper** bound: `xmax = xguess`
- `diff == 0` means the guess is already on the correct coordinate in that axis → **don't adjust** that bound at all

---

## Exclusive bounds and the +1 rule

I noticed that the search occasionally didn't exit. This was a problem, as the program was stuck in a loop. This was cause by the logic for the diff - I was always searching in the same bounds. I needed to make the bound exclusive.

When `diff > 0`, setting `xmin = xguess` (without `+1`) means the new lower bound still includes the current guess - a coordinate already known to be wrong. Using `xmin = xguess + 1` makes the bound exclusive, ensuring the search never wastes a guess on a coordinate already ruled out.

Both axes must use the same convention (both exclusive) or the window can get stuck. An asymmetric implementation - where one axis uses exclusive bounds and the other doesn't was causing infinite loops.

---

## rand.IntN range and shifting

So, I had the `min / max` and now I could use that to search a smaller portion of the slice. However, I did not want to search from the beginning of the slice again, so I had to add the xmin to start the search. This meant that we only used the correct part of the slice. Simples!

`rand.IntN(n)` returns a value in `[0, n)`. To get a random value within a window `[xmin, xmax)`:

```go
xguess = rand.IntN(xmax-xmin) + xmin
```

Without `+ xmin`, you always guess from the origin of the slice (index 0) regardless of where the window is.

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

I found one rare case, whereby the first game would guess correctly. This meant that the number of guesses was 0. This led to a divide by zero, which caused an error to be printed (the program still ran, I just saw the `NaN` value printed). To fix this, I had to add a guard for the case where `randomGuesses == 0`:

```go
if randomGuesses == 0 {
    return 100.0
}

```

```
efficiency = (randomGuesses - midpointGuesses) / randomGuesses * 100
```

This expresses how many fewer guesses midpoint needed as a percentage of random's total. Watch out for `NaN`: if `randomGuesses == 0` (the first guess was correct), this divides by zero. Go's float arithmetic returns `NaN` rather than panicking, so it must be guarded explicitly

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

- `go func()` spawns a goroutine - a lightweight concurrent function
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

Go passes integers by value, not by reference. A function that takes `min, max int` and modifies them only modifies its own local copies - the caller's variables are unchanged. To propagate changes back, either return the new values (idiomatic) or pass pointers. Multiple return values are the preferred Go approach:

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
