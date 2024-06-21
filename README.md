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
    t.run()
}
```

Each test in the map passed to `registerTests` must have this signature:
```go
type TestFn* = fn (T: ^Context): std::Err
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
As of now, the result must be checked for a non-zero error code and returned manually.
The last thing you should do is to return `T.end()` or `T.pass()` to mark the end of the testing function.

## API

### Exported types/values

#### `const VERSION`

The current version number of the library, formatted as specified by the
[Semantic Versioning Specification](https://semver.org/).

#### `newContext(color: bool = false): ^Context`

Sets up and returns a new, heap-allocated `Context`.
`color` dictates whether the tests should print with color or not.

#### `type TestFn = fn (T: ^Context)`

The signature of a test function.

#### `type Context = struct { ... }`

The definition of a testing context.

#### `type Result = struct { result: std::Err; time: real }`

The results of a test.

### Context functions

> [!IMPORTANT]
> All of these are aliased to a specific `Context` (`(c: ^Context)`).

#### `registerTests(tests: map[str]TestFn)`

Registers various tests consecutively.
Will throw a fatal error if the name is already registered with this context.

#### `registerTest(name: str, test: TestFn)`

Registers a single new test.
Will throw a fatal error if the name is already registered with this context.

#### `run(quitIfErr: bool = true): (bool, map[str]Result)`

Runs each of the tests registered to this context, and returns:
- Whether any single one of them had an error
- A map with the results of each one of the tests, with the key being the name of the test.

`quitIfErr` exits the application after every test is run, if any single one of them has any errors.

### Test suite

> [!IMPORTANT]
> All of these are aliased to a specific `Context` (`(c: ^Context)`).
> They will all throw a fatal error if called outside of a test function.

#### `fail(msg: str, code: int = -1)`

Marks this test as failed. You should immediately return once calling this.

#### `pass(msg: str)`

Marks this test as passing. You should immediately return once calling this.

#### `assert.isTrue(cond: bool, msg: str = ""): bool`

Asserts that `cond` is true. If the resulting `bool` is false, the caller should return immediately.
If `msg` is not `""`, prints an extra reason alongside the error.

#### `assert.isFalse(cond: bool, msg: str = ""): bool`

Asserts that `cond` is false. Everything else from `assert.isTrue` applies.
(Currently, this is literally just a call to `assert.isTrue` with the condition inverted.)

#### `assert.isOk(e: std::Err, msg: str = ""): std::Err`

Asserts that `e`'s code is 0. If the resulting `bool` is false, the caller should return immediately.
If `msg` is not `""`, prints an extra reason alongside the error. If it is, it defaults to `e`'s error message.

## Licensing

This library is licensed under the [BSD 2-Clause](./LICENSE) license. See LICENSE for details.
