# toast

A small testing framework with no dependencies written in
the [Umka](https://github.com/vtereshkov/umka-lang) programming language.

- Pretty simple; around 200-250 sloc, easy to read and modify/extend
- Carries timing information and keeps track of test results
- Can print testing information in a slick and straightforward format, or a verbose but insightful one

## Usage

### Creating a new test suite

The simplest (and currently, only) way to create a test suite is to make a new `Context`,
register one or more tests to it and then run it:

```go
fn main() {
    T := toast::newContext()
    T.registerTests({
        "test 1": test1,
        "test 2": test2,
    })
    T.registerTest("test 3", test3)
    T.run()
}
```

Each test in the map passed to `registerTests` must have this signature:
```go
type TestFn* = fn (T: ^Context)
```

### Test functions

A sample test function goes like this:

```go
import "std.um"

fn sampleTest(T: ^toast::Context) {
    recipes := getRecipes()

    res := T.assert.isTrue(
        validkey(recipes, "specialDish"),
        "the path to the special dish isn't listed"
    )

    if !res { return }

    f, e := std::fopen(recipes["specialDish"], "rb")
    if !T.assert.isOk(e) { return }

    // ... do stuff with file ...

    std::fclose(f)
}
```

You set up any needed modules and variables, and then call
one of the methods in the `.assert` struct to check for any conditions.
As of now, the result must be checked for a `false` value and the test returned from manually.

### Custom test functions

You can also make custom test functions by, either mixing and matching assertions,
or encoding your own logic with the help of the `fail` and `pass` functions.

For example, here's how an `assertEqualTypes` function could be written
using custom logic for some specific interface `T`:

```go
fn assertEqualTypes(T: ^toast::Context, a, b: T): bool {
    if !selftypeeq(a, b) {
        T.fail("expected a and b to have the same type")
        return false
    }

    return true
}
```

It can then be used as any other test function:

```go
// ...

// this will pass
if !assertEqualTypes(T, 1, 2) { return }
// this will fail
if !assertEqualTypes(T, 1, "aeiou") { return }

// ...
```

Here's also a pretty pointless equal values assertion implemented
with the `assert.isTrue` call (for any two comparable types `T` and `U`):

```go
fn assertEqual(T: ^toast::Context, a: T, b: U): bool {
    return T.assert.isTrue(a == b, "expected a and b to be equal")
}
```

## Licensing

This library is licensed under the [BSD 2-Clause](./LICENSE) license. See LICENSE for details.
