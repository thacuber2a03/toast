# tests.um
A small testing framework with no dependencies written in
the [Umka](https://github.com/vtereshkov/umka-lang) programming language.

## Usage

### Creating a new test suite

The simplest (and currently, only) way to create a test suite is to make a new Context,
register one or more tests to it and then run it:

```go
fn main() {
    t := tests::newContext()
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

fn sampleTest(T: ^tests::Context): std::Err {
    var e: std::Err

    recipes := getRecipes()

    e = T.assert.isTrue(
        validkey(recipes, "specialDish"),
        "the path to the special dish isn't listed"
    )
    if e.code != 0 { return e }

    f, e := std::fopen(recipes["specialDish"], "rb")
    e = T.assert.isOk(e)
    if e.code != 0 { return e }

    return T.end()
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

Sets up and returns a new `Context`.
`color` dictates whether the tests should print with color or not.

#### `type TestFn = fn (T: ^Context): std::Err`

The signature of a test function.

#### `type Context = struct { ... }`

The definition of a testing context.

#### `type Result = struct { result: std::Err; time: real }`

The result of each test.

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

#### `fail(msg: str, code: int = -1): std::Err`

Returns an `std::Err` marking a test as failed. You should immediately return this back to the caller.

#### `pass(msg: str): std::Err`

Returns an `std::Err` marking a test as passing. You should immediately return this back to the caller.

#### `end(): std::Err`

Returns an `std::Err` marking the end of a test. This should be the last call to the test suite, and should be immediately returned to the caller.

#### `assert.isTrue(cond: bool, msg: str = ""): std::Err`

Asserts that `cond` is true. If the resulting `std::Err`'s code is *not* 0, it should be immediately returned to the caller.
If `msg` is not `""`, prints an extra reason alongside the error.

#### `assert.isFalse(cond: bool, msg: str = ""): std::Err`

Asserts that `cond` is false. Everything from `assert.isTrue` applies.
(Currently, this is literally just a call to `assert.isTrue` with the condition inverted.)

#### `assert.isOk(e: std::Err, msg: str = ""): std::Err`

Asserts that `e`'s code is 0. If the resulting `std::Err`'s code is *not* 0, it should be immediately returned to the caller.
If `msg` is not `""`, prints an extra reason alongside the error. If it is, it defaults to `e`'s error message.

## Licensing

This library is licensed under the [BSD 2-Clause](./LICENSE) license. See LICENSE for details.
