---
title: Data Structures and Functions
---

When storing multiple values of the same type, start by deciding whether you need to find elements by **position** or by **key**. Arrays and slices work with ordered sequences, while maps associate data with a key. Use `bufio` to read text input and small functions to organize the logic that processes it.

## Arrays

An array holds a fixed number of elements of the same type. The length is part of the type, so `[3]int` and `[4]int` are different types. In `var scores [3]int`, each element starts with the zero value of `int`. Use `[3]int{18, 16, 20}` to initialize the elements. With `[...]int{18, 16, 20}`, the compiler infers the length.

Indexes start at zero. The last valid index is `len(scores)-1`. Assigning an array or passing it to a function **copies the array value**.

```go
scores := [...]int{18, 16, 20}
updated := scores
updated[0] = 19

fmt.Println(scores[0], updated[0]) // 18 19
```

Choose a `range` form depending on what you need from the array or slice.

- `for i, value := range scores` gives you the index and the value.
- `for _, value := range scores` keeps just the value.
- `for i := range scores` gives you just the index. Assign to `scores[i]` to update the original element.

Use `for i := 0; i < len(scores); i++` when you need to control the counter yourself. A multidimensional array is an array of arrays. For example, `var matrix [2][3]int` has two rows of three elements, and `matrix[0][1]` is the second element of the first row. Array elements are stored contiguously and index access takes constant time, `O(1)`. The compiler decides whether to allocate an array on the stack or heap based on how it is used.

## Slices

A slice is a variable-length view of an **underlying array**. Its slice header points to where the elements start and holds the length and capacity. `len(s)` counts the current elements, while `cap(s)` counts the slots available from that starting point in the underlying array. Passing a slice to a function copies the header and shares access to the underlying elements.

| Creation | Result |
| --- | --- |
| `[]int{10, 20}` | Two initialized elements |
| `make([]int, 3)` | Length and capacity of 3, with three zero elements |
| `make([]int, 0, 3)` | Length of 0 and capacity of 3, ready for appending |
| `var s []int` | A `nil` slice with zero length and capacity that you can append to |

In `values[low:high]`, the slice includes `low` and stops before `high`. `values[:2]` takes the first two elements, `values[2:]` takes the elements from index 2 to the end, and `values[:]` covers all current elements.

Assign the result of `append` back to the slice variable. When capacity is sufficient, `append` uses the existing underlying array. When the slice needs more room, it allocates a new array and copies the existing elements.

```go
scores := [3]int{10, 20, 30}
window := scores[1:] // len=2, cap=2
window[0] = 25
fmt.Println(scores) // [10 25 30]

window = append(window, 40) // Grow past capacity.
window[0] = 99
fmt.Println(scores) // [10 25 30]
fmt.Println(window) // [99 30 40]
```

`append(make([]int, 3), 5)` produces `[0 0 0 5]` because the three initial elements already count toward the length. Assign by index to change one of those elements. To append all the elements from another slice, use `values = append(values, more...)`. Slices are the usual choice for lists whose length is determined or changes at runtime.

## Maps

A map of type `map[K]V` associates unique keys of type `K` with values of type `V`. Use a map to look up data by name or ID, count occurrences, or store key-value pairs. The map hashes each key to narrow where it searches. Typical key operations take `O(1)` time on average.

Create a writable map with `make(map[string]int)` or a literal such as `map[string]int{"pens": 0}`. The zero value of a map is `nil`. Reading from a nil map returns the value type's zero value. Writing to it panics.

```go
stock := map[string]int{"pens": 0}
stock["notebooks"] = 12

count, ok := stock["pens"]
fmt.Println(count, ok) // 0 true

count, ok = stock["markers"]
fmt.Println(count, ok) // 0 false

delete(stock, "notebooks")
fmt.Println(len(stock)) // 1
```

The **comma-ok** form `value, ok := m[key]` distinguishes a missing key from a present key with a zero value. Use `m[key] = value` to insert or update an entry. In a `map[string]int` initialized with `make` or a literal, `counts[name]++` starts a new key's count at one. Deleting a missing key has no effect. `for key, value := range m` gives you each entry's key and value. Iteration order is unspecified.

Map key types must be comparable. Numbers, strings, booleans, pointers, and channels qualify. Arrays and structs qualify when all their elements or fields are comparable. An interface key must hold a comparable concrete value. Slices, maps, and functions cannot be key types.

## Reading input with `bufio`

A buffer keeps a chunk of input in memory temporarily so small reads can reuse it. `Scanner` is a straightforward way to read lines that can contain spaces, such as a teacher's full name. `Text()` gives you the current line without its line ending. Check `Err()` when the loop finishes to detect read failures.

```go
scanner := bufio.NewScanner(os.Stdin)
for scanner.Scan() {
	fmt.Println(scanner.Text())
}
if err := scanner.Err(); err != nil {
	fmt.Fprintln(os.Stderr, err)
}
```

Use `Reader` for a custom delimiter or more control over reading. `ReadString('\n')` returns the line including the delimiter when it finds one. A final line without a line ending can arrive with `io.EOF`. Process any data returned in that case.

```go
reader := bufio.NewReader(os.Stdin)
line, err := reader.ReadString('\n')
if err != nil && err != io.EOF {
	fmt.Fprintln(os.Stderr, err)
	return
}
if line != "" {
	fmt.Println(strings.TrimRight(line, "\r\n"))
}
```

`TrimRight` removes the line-ending characters from the end of the string and preserves spaces inside the line. `Scanner` suits line-by-line or word-by-word processing. `Reader` offers finer control with methods such as `Peek`, which lets you see upcoming bytes without consuming them.

## Functions

Define a function with `func` and give it a specific job. In `func add(a, b int) int`, `a` and `b` are parameters, and the final `int` is the return type. The values `2` and `3` in `add(2, 3)` are arguments. Consecutive parameters of the same type can share a type declaration. A function that returns no values omits the return type.

A function can return a result alongside a success flag or an `error`. This function signals to the caller that it cannot compute an average for an empty collection.

```go
func average(sum, count int) (float64, bool) {
	if count == 0 {
		return 0, false
	}
	return float64(sum) / float64(count), true
}
```

In `value, ok := average(15, 2)`, the result is `7.5` and `ok` is `true`. The caller should check the flag before using the result. You can name return values, as in `(area int, perimeter int)`. A bare `return` (a naked return) returns the current values of those variables. In longer functions, `return area, perimeter` makes the exit path easier to read.

A variadic parameter goes last in the parameter list and accepts a variable number of arguments. Inside the function, you can iterate over it as a slice.

```go
func sum(values ...int) int {
	total := 0
	for _, value := range values {
		total += value
	}
	return total
}
```

`sum(1, 2, 3)` returns `6`. If the data is already in a slice called `values`, call `sum(values...)` to pass its elements.

## Function values and closures

Functions in Go are first-class values. You can store one in a variable, pass it to another function, or return it from a function. An anonymous function has no declared name. You can immediately call one with `()`, as in `func() { fmt.Println("Ready") }()`.

```go
func apply(n int, op func(int) int) int {
	return op(n)
}

func multiplier(factor int) func(int) int {
	return func(n int) int {
		return n * factor
	}
}
```

`double := multiplier(2)` returns a function that can still access `factor` after `multiplier` returns. A function together with the variables it captures is a **closure**. `apply(5, double)` returns `10`. A function such as `apply` or `multiplier` that takes or returns a function is a higher-order function.

## Running cleanup with `defer`

`defer` schedules a call to run when the current function exits. For resource cleanup, place it immediately after successfully acquiring the resource, as in `defer file.Close()` after opening a file. Multiple deferred calls run in reverse order (LIFO).

```go
func finish() {
	defer fmt.Println("first defer")
	defer fmt.Println("second defer")
	fmt.Println("work")
}
```

Calling `finish()` prints `work`, then `second defer`, then `first defer`.
