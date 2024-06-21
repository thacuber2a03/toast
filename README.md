# toast

A small testing framework with no dependencies written in
the [Umka](https://github.com/vtereshkov/umka-lang) programming language.

- Pretty simple; less than 200 lines, easy to read and modify/extend
- Carries timing information and keeps track of test results

## Usage

### Creating a new test suite

The simplest (and currently, only) way to create a test suite is to make a new `Context`,
register one or more tests to it and then run it:

```go
fn main() {
    t := toast::newContext()
    t.registerTests({
        "test 1": test1,
        "test 2": test2,
    })
    t.registerTest("test 3", test3)
    t.run()
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

## Licensing

This library is licensed under the [BSD 2-Clause](./LICENSE) license. See LICENSE for details.
