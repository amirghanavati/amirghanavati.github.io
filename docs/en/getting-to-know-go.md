---
title: Introduction to Go
---

This chapter builds a command-line Go program and introduces the tools it needs to store data, make decisions, repeat work, and read input. To run the examples, install Go on your system first.

## Your first program

In an empty directory, run `go mod init hello`. A module groups related packages that are versioned together. The command creates `go.mod`, which records the module path, Go version, and project dependencies.

Save the following as `main.go`. The `main` package produces an executable, and execution starts in the `main` function. The `import` declaration makes the standard package `fmt` available.

```go
package main

import "fmt"

func main() {
	fmt.Println("Hello, World!")
}
```

| Command | When to use it |
| --- | --- |
| `go run main.go` | Compile and run the code for a quick check. It leaves no permanent executable in the project directory. |
| `go build` | Build the executable for this module. Run it with `./hello` on Linux or macOS, or with `.\hello.exe` in PowerShell. |

The built-in `println` is available without an import for simple examples. Use `fmt` for console output and formatting.

## Variables, constants, and types

`var` declares a variable with a specific type. If you omit the initializer, Go supplies the type's **zero value**. With an initializer, the compiler can infer the type. The short declaration `:=` is available inside functions. Use `const` for a compile-time value that stays fixed.

```go
var earthWeight int // 0 before assignment
var name = "Helia"   // string
earthWeight = 80
const gravityRatio = 0.3783
equivalentWeight := float64(earthWeight) * gravityRatio
fmt.Printf("%s: %.3f\n", name, equivalentWeight) // Helia: 30.264
```

Declare multiple variables of the same type together with syntax such as `var x, y int`. Use `var (...)` and `const (...)` blocks to group related declarations.

| Type | Use | Zero value |
| --- | --- | --- |
| `int`, `int8` through `int64` | Signed integers | `0` |
| `uint`, `uint8` through `uint64` | Nonnegative integers | `0` |
| `float32`, `float64` | Floating-point numbers with different precision | `0` |
| `bool` | `true` or `false` for conditions | `false` |
| `string` | An immutable sequence of bytes, typically interpreted as UTF-8 for text | `""` |
| `byte` | Alias for `uint8`, useful for a single byte such as ASCII `'A'` | `0` |
| `rune` | Alias for `int32`, used for a Unicode code point such as `'ش'` | `0` |

The number in a type such as `int8` indicates its bit width and affects the range it can hold. In a double-quoted string, `\n` becomes a newline. A raw string uses backticks, can span multiple lines, and preserves `\n` as written.

Go is statically typed. To calculate with an `int` variable and a `float64` value, convert one operand explicitly so the types match, as in the weight example. Converting a floating-point value to an integer drops the fractional part. To convert text to a number, use the standard package `strconv`.

## Control flow

### Conditions

`if` evaluates a boolean condition and always uses braces for its body. In an `if / else if / else` chain, the first true condition determines which branch runs. You can put a short statement before the condition. A variable declared there with `:=` is available throughout that `if` and its `else` branches.

```go
if num := 9; num < 0 {
	fmt.Println("Negative")
} else if num < 10 {
	fmt.Println("One digit")
}
```

Use `switch value` to compare a value against several options. Group values with the same behavior in one branch, such as `case "+", "-", "*", "/":`. The optional `default` runs when no case matches. Go exits the switch after running a matching case. Use `fallthrough` to run the next case body without testing its condition. In a switch without an expression, Go checks the conditions in order and runs the first true case.

### Loops

| Loop | When to use it |
| --- | --- |
| `for i := 1; i <= 5; i++` | Initialize once, check the condition before each iteration, and update after each iteration. |
| `for n > 0` | Repeat while a condition holds. Update `n` in the loop body. |
| `for { ... }` | Repeat continuously. Provide an exit path, such as the `exit` command in a CLI. |

`break` ends the loop. `continue` skips the rest of the current iteration. This example prints odd numbers below 8.

```go
for i := 1; i < 15; i++ {
	if i >= 8 {
		break
	}
	if i%2 == 0 {
		continue
	}
	fmt.Println(i)
}
```

## Packages, naming, and scope

A package groups related `.go` files in one directory. Use `import` to bring another package into scope and qualify exported members with the package name, as in `fmt.Println`. Multiple imports can share an `import (...)` block.

- Use camelCase for multiword unexported names, such as `defaultHost`. A package-level name that starts with an uppercase letter is exported and accessible from other packages, such as `MaxConnections`.
- Unused local variables and imports cause compilation errors. The blank identifier `_` discards a value deliberately. For example, `_, err := fmt.Scan(&name)` discards the count and keeps the error. An import prefixed with `_` is used for the package's side effects, such as initialization.
- Shadowing occurs when a new variable in an inner block has the same name as an outer variable. If you declare `level := "Outer"` outside a block and `level := "Inner"` inside it, the outer value is still `"Outer"` after the block. Use `=` to update an existing variable.

## Input and output with `fmt`

| Function | Behavior |
| --- | --- |
| `Println` | Prints arguments separated by spaces and adds a newline. |
| `Print` | Leaves the cursor on the same line unless the output includes a newline. Useful for input prompts. Include any space you need between two strings in the text. |
| `Printf` | Prints using a format string. Include `\n` in the format to add a newline. |
| `Sprintf` | Returns the formatted text as a `string` for later use. |
| `Scan` | Splits input into values at spaces, tabs, or newlines. A string destination typically receives one word. |
| `Scanln` | Scans separate values and stops at a newline. |
| `Scanf` | Matches input against a format, such as three numbers separated by hyphens. |

### Format verbs

A format verb tells `Printf` or `Sprintf` how to display a value. Each example shows the input value on the left and the resulting text on the right.

| Verb | Use | Example |
| --- | --- | --- |
| `%v` | Value in its default format | `3.14 → 3.14` |
| `%T` | Type of the value | `42 → int` |
| `%d` | Integer in decimal | `42 → 42` |
| `%b` | Integer in binary | `10 → 1010` |
| `%o` | Integer in octal | `10 → 12` |
| `%x` | Integer in lowercase hexadecimal | `255 → ff` |
| `%X` | Integer in uppercase hexadecimal | `255 → FF` |
| `%f` | Floating-point number with six decimal places | `3.14159 → 3.141590` |
| `%.2f` | Floating-point number rounded to two decimal places | `3.14159 → 3.14` |
| `%s` | String text | `"Go" → Go` |
| `%q` | Double-quoted string with special characters escaped | `"Go" → "Go"` |
| `%c` | Character for a Unicode code point | `65 → A` |
| `%t` | Boolean as `true` or `false` | `true → true` |
| `%p` | Address stored in a pointer, in hexadecimal | `&n → 0x…` |

### Checking input

Pass the address of each destination variable with `&` so the scan function can store the input in it. These functions return the number of values read and an error. Check `err` before using the input. This example scans the structure of a date. Validate the month and day separately.

```go
var year, month, day int
if _, err := fmt.Scanf("%d-%d-%d", &year, &month, &day); err != nil {
	fmt.Println("Use year-month-day")
	return
}
fmt.Printf("Year=%d, Month=%d, Day=%d\n", year, month, day)
```

Input separators and spacing matter with `Scanf`. For example, `2025/11/08` does not match the hyphenated format. Use `Fscanf` and `Fprintf` to read from another source or write to another destination.
