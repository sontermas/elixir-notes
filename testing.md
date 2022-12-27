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

Here are some ways to run tests:
| Command | Description |
| ------- | ----------- |
| `mix test` | Run all the project’s tests. |
| `mix test test/file_test.exs` | Run all the file's tests. |
| `mix test test/file_test.exs:10` | Run the test case at line 10. |

When you run the `mix test` task, it starts the current application, loads up `test/test_helper.exs` and then requires all files matching the `test/**/_test.exs` pattern in parallel.

Run `mix help test` for more information.

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

The following table lists macros that can be used to make assertions.  We will look at examples of these in subsequent sections.
| Macro | Description |
| -------- | ----------- |
| `assert` | Test that the expression is `true`. Will only pass if the expression is _truthy_. |
| `refute` | Test that the expression is `false`. Will only pass if the expression if _false_ or _nil_.zz |
| `assert_raise`  | Assert that an error has been raised. |
| `assert_receive` | Test whether a message has been received. You can specify a timout. |
| `assert_received` | Test whether a message has been received. Will not wait for messages. |
| `capture_io` | Used to capture an application's output. |
| `capture_log` | Used to capture output to `Logger`. |

## Test Setup
In some instances it may be necessary to perform setup before our tests. To accomplish this we can use the `setup` and `setup_all` macros. `setup` will be run before each test and `setup_all` once before the suite. It is expected that they will return a tuple of `{:ok, state}`, the state will be available to our tests.

## Test Organization
You basically have two macros: `test` and `describe` to organize your testing scenarios and examples.

### `describe`
You can use the `describe` macro to group tests that share a common setup. The creators of ExUnit made the decision to only allow one level of grouping, so you can’t nest `describe` blocks.  A guide that can help drive your design is whether or not your functions have a common setup.

Example:
```elixir
defmodule YourApp.YourModuleTest do
  use ExUnit.Case
  
  describe "thing_to_do/1" do
    test "it returns :ok, calls the function if the key is correct"
    test "it does not call the function if the key is wrong"
  end
end
```

### `setup`
The code in a `setup` block will be executed before each test inside of its scope. Since the code has the same name as the stage in the test design, we'll call the executable code a setup block while referring to the stage of testing as setup.

A setup block returns a value that makes sense for what comes after it. If the setup is to set state elsewhere and none of the values are useful in the tests following it, the return value can be as simple as `:ok`. 

More often, the return value from a setup block is passed to the tests it precedes. The common practice is to have the setup block return a map of values that can be used in the subsequent test. When testing purely functional code, this is often the only time you would use a setup block, since there's no shared state to set up. Let's look at an example of a setup block where no state is changed but we've decided that we don't want to redefine the same anonymous functions for each test.
```elixir
setup do
  function_to_not_call = fn ->
    flunk("this function should not have been called")
  end

  function_to_call = fn -> send(self(), :function_called) end
  %{bad_function: function_to_not_call, good_function: function_to_call}
end
```

Accepting values from a setup in your tests requires a slight modification to your test definition; you'll need to add a pattern as a second parameter to your test allows you to pull in anything passed from the setup.
```elixir
test "does not call the function if the key is wrong", %{bad_function: bad_function} do
  assert {:error, _} = YourModule.thing_to_do(:bad_first_arg, bad_function)
end
```

You can always just use a variable and pull what you need from it when you need it.
```elixir
test "does not call the function if the key is wrong", context do
  assert {:error, _} = YourModule.thing_to_do(:bad_first_arg, context.bad_function)
end
```

## Test Mocks

## Running Tests in a Non-Mix Environment


## References
- [ExUnit Docs](https://hexdocs.pm/ex_unit/ExUnit.html)
- [Elixir School - Testing](https://elixirschool.com/en/lessons/testing/basics)
