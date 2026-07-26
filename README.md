# ltestlib

[![LuaRocks](https://img.shields.io/luarocks/v/luau-project/ltestlib?label=LuaRocks&color=2c3e67)](https://luarocks.org/modules/luau-project/ltestlib)

Yes, this is yet another test library for Lua (&ge; 5.1), but with very specific goals.

## Goals

### General

The main intent of [ltestlib.lua](./ltestlib.lua) is to provide a minimal and rudimentary test library for Lua projects without any dependencies;

### Specific

This library was created to be used in a drag-and-drop style. This means that you can drop [ltestlib.lua](./ltestlib.lua) on your own project, then start to use it immediately.

* No need to give credits;
* No warranties of any kind;
* Use at your own risk.

## Table of Contents

* [Usage](#usage)
* [How to test multiple files](#how-to-test-multiple-files)
* [Properties](#properties)
    * [version](#version)
* [Methods](#methods)
    * [assert_equal](#assert_equal)
    * [assert_false](#assert_false)
    * [assert_not_equal](#assert_not_equal)
    * [assert_throws](#assert_throws)
    * [assert_true](#assert_true)
    * [execute](#execute)
    * [finish](#finish)
    * [log](#log)
    * [logfmt](#logfmt)
    * [new_test](#new_test)
    * [reset](#reset)
    * [summary](#summary)
* [License](#license)
* [History](#history)

## Usage

1. Create a directory `somedir`, then copy [ltestlib.lua](./ltestlib.lua) to `somedir` (change `somedir` to a suitable name)
2. Add a file `addition.lua` to be tested to the directory `somedir` containing the following code:

    ```lua
    local function add(a, b)
        return a + b
    end

    local function increment(v)
        return add(v, 1)
    end

    local function wrongincrement(v)
        return add(v, 2)
    end

    local function evenbug(v)
        if ((v % 2) == 0) then
            error("Even numbers are not allowed.")
        end
        return v
    end

    return {
        add = add,
        increment = increment,
        wrongincrement = wrongincrement,
        evenbug = evenbug
    }
    ```

3. Add a file `test.lua` -- the test driver -- to test `addition.lua` using `ltestlib.lua`:

    ```lua
    local ltestlib = require("ltestlib")
    local addition

    -- a helper table to store test names
    local testnames = {}

    -- create a test to ensure 'require' works
    testnames.n1 = "addition module should return a table"
    ltestlib.new_test(testnames.n1, function()
        addition = require("addition")
        ltestlib.assert_equal(testnames.n1, "table", type(addition))
    end)

    -- create a test to ensure 'add' works
    testnames.n2 = "add method should sum correctly two numbers"
    ltestlib.new_test(testnames.n2, function()
        ltestlib.assert_equal(testnames.n2, 3, addition.add(1, 2))
        ltestlib.assert_equal(testnames.n2, 3, addition.add(2, 1))
        ltestlib.assert_equal(testnames.n2, 1, addition.add(-1, 2))
        ltestlib.assert_equal(testnames.n2, 1, addition.add(2, -1))
        ltestlib.assert_equal(testnames.n2, -3, addition.add(-1, -2))
        ltestlib.assert_equal(testnames.n2, -3, addition.add(-2, -1))
        ltestlib.assert_equal(testnames.n2, -1, addition.add(1, -2))
        ltestlib.assert_equal(testnames.n2, -1, addition.add(-2, 1))
    end)

    -- create a test to ensure 'increment' works
    testnames.n3 = "increment method should add one to a number"
    ltestlib.new_test(testnames.n3, function()
        ltestlib.assert_equal(testnames.n3, 1, addition.increment(0))
        ltestlib.assert_equal(testnames.n3, 2, addition.increment(1))
        ltestlib.assert_equal(testnames.n3, 0, addition.increment(-1))
    end)

    -- create a test thinking that 'wrongincrement' works like 'increment'
    testnames.n4 = "wrongincrement method should add one to a number"
    ltestlib.new_test(testnames.n4, function()
        ltestlib.assert_equal(testnames.n4, 1, addition.wrongincrement(0))
        ltestlib.assert_equal(testnames.n4, 2, addition.wrongincrement(1))
        ltestlib.assert_equal(testnames.n4, 0, addition.wrongincrement(-1))
    end)

    -- create a test to ensure 'evenbug' returns the provided param when param is odd
    testnames.n5 = "evenbug method should return the provided param when param is an odd number"
    ltestlib.new_test(testnames.n5, function()
        ltestlib.assert_equal(testnames.n5, -1, addition.evenbug(-1))
        ltestlib.assert_equal(testnames.n5, 1, addition.evenbug(1))
        ltestlib.assert_equal(testnames.n5, 3, addition.evenbug(3))
    end)

    -- create a test to ensure 'evenbug' throws an error once the provided param is even
    testnames.n6 = "evenbug method should throw an error if the provided param is an even number"
    ltestlib.new_test(testnames.n6, function()
        ltestlib.assert_throws(testnames.n6, function() addition.evenbug(0) end)
        ltestlib.assert_throws(testnames.n6, function() addition.evenbug(2) end)
        ltestlib.assert_throws(testnames.n6, function() addition.evenbug(4) end)
        ltestlib.assert_throws(testnames.n6, function() addition.evenbug(-2) end)
        ltestlib.assert_throws(testnames.n6, function() addition.evenbug(-4) end)
    end)

    -- execute the tests
    ltestlib.execute()

    -- collect statistics
    -- and print the summary of execution
    ltestlib.summary()

    -- finish all the tests
    -- returning the execution code
    ltestlib.finish()
    ```

4. Change directory to `somedir` and run the tests:

    ```bash
    cd somedir
    lua test.lua
    ```

5. Review the test output

    ```
    ======================== Start of Execution ========================

    [1][Passed]: addition module should return a table
    [2][Passed]: add method should sum correctly two numbers
    [3][Passed]: increment method should add one to a number
    [4][Failed]: wrongincrement method should add one to a number
        [1] EQUAL assertion failed: Expected (1) is not equal to what we Got (2)
        [2] EQUAL assertion failed: Expected (2) is not equal to what we Got (3)
        [3] EQUAL assertion failed: Expected (0) is not equal to what we Got (1)
    [5][Passed]: evenbug method should return the provided param when param is an odd number
    [6][Passed]: evenbug method should throw an error if the provided param is an even number

    ========================= End of Execution =========================


    ========================= Start of Summary =========================

    Tests Executed: 6
        Tests Passed: 5 (83.3%)
        Tests Failed: 1 (16.7%)

    Assertions Executed: 23
        Assertions Passed: 20 (87.0%)
        Assertions Failed: 3 (13.0%)

    ========================== End of Summary ==========================
    ```

## How to test multiple files

It is up to the user how to define the tests through `ltestlib.new_test`.

The important part is to centralize the three calls

```lua
    -- execute the tests
    ltestlib.execute()

    -- collect statistics
    -- and print the summary of execution
    ltestlib.summary()

    -- finish all the tests
    -- returning the execution code
    ltestlib.finish()
```

in the main test driver (`test.lua`).

## Properties

### version

* Description: the version of this library.
* Signature: `ltestlib.version`
* Return (string): a string containing the library version.

## Methods

### assert_equal

* Description: asserts two values are equal.
* Signature: `ltestlib.assert_equal(name, expected, value[, msg])`
* Parameters:
    * `name` (string): the name of the test
    * `expected` (any): the expected value
    * `value` (any): the provided value under test
    * `msg` (string | nil): optional message string
* Return (void).

### assert_false

* Description: asserts the expected value is `false`.
* Signature: `ltestlib.assert_false(name, value[, msg])`
* Parameters:
    * `name` (string): the name of the test
    * `value` (any): the provided value under test
    * `msg` (string | nil): optional message string
* Return (void).

### assert_not_equal

* Description: asserts two values are not equal.
* Signature: `ltestlib.assert_not_equal(name, expected, value[, msg])`
* Parameters:
    * `name` (string): the name of the test
    * `expected` (any): the expected value to be different
    * `value` (any): the provided value under test
    * `msg` (string | nil): optional message string
* Return (void).

### assert_throws

* Description: asserts the execution of a test throws an error.
* Signature: `ltestlib.assert_throws(name, f[, msg])`
* Parameters:
    * `name` (string): the name of the test
    * `f` (function): the callback function to execute during the test execution
    * `msg` (string | nil): optional message string
* Return (void).

### assert_true

* Description: asserts the expected value is `true`.
* Signature: `ltestlib.assert_true(name, value[, msg])`
* Parameters:
    * `name` (string): the name of the test
    * `value` (any): the provided value under test
    * `msg` (string | nil): optional message string
* Return (void).

### execute

* Description: perform the execution of tests defined by each call to `new_test`.
* Signature: `ltestlib.execute()`
* Parameters: (void)
* Return (void).

### finish

* Description: finish the tests returning 1 as exit code when any of the tests / assertions failed.
* Signature: `ltestlib.finish()`
* Parameters: (void)
* Return (void).

### log

* Description: writes a message to the test log concatenating arguments by a tab character.
* Signature: `ltestlib.log(name[, ...])`
* Parameters:
    * `name` (string): the name of the test
    * `...` (vararg): optional parameters
* Return (void).

### logfmt

* Description: writes a formatted message to the test log through `string.format`.
* Signature: `ltestlib.logfmt(name, msg[, ...])`
* Parameters:
    * `name` (string): the name of the test
    * `msg` (string): mandatory message pattern to format the message
    * `...` (vararg): optional parameters to format the message
* Return (void).

### new_test

* Description: defines a new test.
* Signature: `ltestlib.new_test(name, f)`
* Parameters:
    * `name` (string): the name of the test
    * `f` (function): the callback function to execute during the test execution
* Return (void).

### reset

* Description: resets the library to a clean state, removing any tests defined previously.
* Signature: `ltestlib.reset()`
* Parameters: (void)
* Return (void).

### summary

* Description: collects summary statistics for the execution of tests and print the summary review.
* Signature: `ltestlib.execute()`
* Parameters: (void)
* Return (table): a table, described below, containing statistics for the tests ran

    | Field | Type of the value | Description |
    |---|---|---|
    | `total_execution_failed` | `number` | The number of tests which failed to run correctly (failed `pcall`). |
    | `total_tests_passed` | `number` | The number of passed tests. |
    | `total_tests_failed` | `number` | The number of tests which ran correctly, but failed. |
    | `total_tests_passed_ratio` | `number` | The pass ratio (0 - 1) of tests. |
    | `total_tests_failed_ratio` | `number` | The failed ratio (0 - 1) of tests. |
    | `total_assertions_passed` | `number` | The number of passed assertions. |
    | `total_assertions_failed` | `number` | The number of failed assertions. |
    | `total_assertions_passed_ratio` | `number` | The ratio (0 - 1) of passed assertions. |
    | `total_assertions_failed_ratio` | `number` | The ratio (0 - 1) of failed assertions. |

## License

This is free and unencumbered software released into the public domain (see [LICENSE.md](./LICENSE.md)). In countries that refuse public domain, then MIT is used (see [LICENSE-MIT.md](./LICENSE-MIT.md)).

## History

See the [changelog](./CHANGELOG.md)