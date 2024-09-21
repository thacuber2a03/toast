# toast

A small testing framework with no dependencies written in
the [Umka](https://github.com/vtereshkov/umka-lang) programming language.

- Pretty simple; less than 300 sloc, easy to read and modify/extend
- Carries timing information and keeps track of test results
<!-- might erase this line -->
- Can print testing information in a slick and straightforward format, or a verbose but in-depth one

## Installation

### Manually

Download the "toast.um" file and place it somewhere in your project.
Then, import it in your test file.

### Through [UmBox](https://umbox.tophat2d.dev)

> [!WARNING]
> This will install the `master` branch version of the box, which might not be stable.

Run `umbox install toast`.
You can also run `umbox init -p toast` to make a new box with Umka and toast preinstalled.

## Usage

### Creating a new test suite

The simplest (and currently, only) way to create a test suite is to make a new `Context`,
register one or more tests to it and then run it:

```go
fn main() {
    T := toast::newContext()
    T.registerTests({
        { name: "test 1", func: test1 },
        { name: "test 2", func: test2 }
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

### Custom assertions

You can also make custom assertion functions by, either mixing and matching assertions,
or encoding your own logic with the help of the `fail` and `pass` functions.

For example, here's how `assert.sameType` function would be written
if it was a custom assertion function:

```go
fn assertSameType(T: ^toast::Context, a, b: any): bool {
    T.startCustom()

    if !selftypeeq(a, b) {
        return T.endCustom(T.fail("expected a and b to have the same type"))
    }

    return T.endCustom(true)
}
```

It can then be used as any other test function:

```go
// ...

// this will pass
if !assertSameType(T, 1, 2) { return }
// this will fail
if !assertSameType(T, 1, "aeiou") { return }

// ...
```

Here's also a pretty pointless equal values assertion implemented
with the `assert.isTrue` call (for any two comparable types `T` and `U`):

```go
fn assertEqual(T: ^toast::Context, a: T, b: U): bool {
    T.startCustom()
    return T.endCustom(T.assert.isTrue(
        a == b, "expected a and b to be equal"
    ))
}
```

## Licensing

This library is licensed under the [BSD 2-Clause](./LICENSE) license. See LICENSE for details.
