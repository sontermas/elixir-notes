# Testing

## ExUnit
Elixir's built-in test framework is `ExUnit` and it includes everything needed to test code. 

This tutorial assumes you'll be using the `Mix` build tool.  If you're not using `Mix`, see [Running Tests in a Non-Mix Environment](#running-tests-in-a-non-mix-environment) at the end of this document for instructions on how to run tests in a non-Mix environment.

When you create a `mix` project, a basic testing structure will be automatically created for you.
```
$ mix new hello
```
```
├── lib
│   └── hello.ex
├── mix.exs
├── README.md
└── test
    ├── hello_test.exs
    └── test_helper.exs
```

A `test` directory will be created containing two files:
- `hello_test.exs` contains our test cases.
- `test_helper.exs` sets the default configuration of all tests. To start, it contains only one line of code: `ExUnit.start`. This line needs to be called before running any tests.

Tests are implemented as Elixir scripts so they need an `.exs` file extension.

## Running Tests
You can use the `mix test` command to run all tests in the project. This will load the `test_helper.exs` file before executing the tests (it is not necessary to require the `test_helper.exs` file in your test files). 

Run `mix help test` for more information.

Here are some ways to run tests:
| Command | Description |
| ------- | ----------- |
| `mix test` | Run all the project’s tests. |
| `mix test test/file_test.exs` | Run all the file's tests. |
| `mix test test/file_test.exs:10` | Run the test case at line 10. |

When you run the `mix test` task, it starts the current application, loads up `test/test_helper.exs` and then requires all files matching the `test/**/_test.exs` pattern in parallel.

## Writing Tests
The following is an example of a basic test file. 

```elixir
defmodule HelloTest do
  use ExUnit.Case
  doctest Hello

  test "greets the world" do
    assert Hello.hello() == :world
  end
end
```

It contains a few things of note:
- `use ExUnit.Case` is a `use` directive that imports macros. A macro is code that writes code through the use of a template.
- `doctest Hello` ignore this for now.
- `test "greets the world" do` defines a test.
- `assert Hello.hello() == :world` uses the `assert` macro to compare the actual result against the expected result.

Here are a few more 
| Macro | Description |
| -------- | ----------- |
| `assert` | Test that the expression is `true`.  |
| `refute` | Test that the expression is `false`. |
| `assert_raise`  | Assert that an error has been raised. |
| `assert_receive` | Test whether a message has been received. You can specify a timout. |
| `assert_received` | Test whether a message has been received. Will not wait for messages. |
| `capture_io` | Used to capture an application's output. |
| `capture_log` | Used to capture output to `Logger`. |

## Test Setup
In some instances it may be necessary to perform setup before our tests. To accomplish this we can use the `setup` and `setup_all` macros. `setup` will be run before each test and `setup_all` once before the suite. It is expected that they will return a tuple of `{:ok, state}`, the state will be available to our tests.

## Test Organization
You basically have two macros: test and describe to organize your testing scenarios and examples.

## Test Mocks

## Running Tests in a Non-Mix Environment


## References
- [ExUnit Docs](https://hexdocs.pm/ex_unit/ExUnit.html)
- [Elixir School - Testing](https://elixirschool.com/en/lessons/testing/basics)
