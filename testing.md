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

The scope of your setup is set by the logical grouping of your tests. If your setup is inside of a describe, it'll be run before the tests grouped inside of that describe. If it's at the top of your test file, outside of any describes - it'll be run before all the tests in that file.

Your setups can build on top of each other. If you have a setup that's common to an entire file but also need a setup for your describe block, the setup in your describe can accept values passed to it from the higher-up describe by adding a pattern to the test's parameters.
```elixir
setup context do
  additional_value = "example value"
  Map.merge(context, %{additional_value: additional_value})
end
```

Setup can accept the name of a function or a list of function names instead of a block. Those functions should conform to a pattern that accepts the existing context (the data passed by previous setup blocks or functions) and return an updated context so that they can be piped. 

As you can see in the following example, with_authenticated_user/1 expects a map of the data from previous setup functions,
adds more data to that map, and then returns that map.
```elixir
def with_authenticated_user(context) do
  user = User.create(%{name: "Bob Robertson"})
  authenticated_user = TestHelper.authenticate(user)
  Map.put(context, :authenticated_user, authenticated_user)
end
```

If you have defined functions that follow that pattern, you can pass a list to `setup` of the functions to execute:
```elixir
setup [:create_organization, :with_admin, :with_authenticated_user]
```

### `setup_all`
You can use `setup_all` if you need a setup step, but you don’t need to repeat it before each test.

Example:
```elixir
setup_all do
  function_to_not_call = fn ->
    flunk("this function should not have been called")
  end

  function_to_call = fn -> send(self(), :function_called) end
  %{bad_function: function_to_not_call, good_function: function_to_call}
end
```

### `on_exit` callback
The `on_exit` callback that lets us organize the teardown stage in a similar way. This callback can be defined inside your setup block or within an individual test.

Example:
```elixir
setup do
  file_name = "example.txt"
  :ok = File.write(file_name, "hello")
  
  on_exit(fn ->
    File.rm(file_name)
  end)

  %{file_name: file_name}
end
```

Notice that the return value of `setup` is still the last line of the block. `on_exit` will return `:ok`, which is likely not useful for your setup.

As with all anonymous functions in Elixir, the function you pass to `on_exit` is a closure, so you can reference any variables defined
before the function is defined. `on_exit` is powerful because even if your test fails spectacularly, the anonymous function will still be executed. This helps guarantee the state of your test environment, no matter the outcome of an individual test.

## Fixtures
It would be nice to be working with data that’s as realistic as possible.

To do this, we can use a fixture file. A simple curl call to the API can provide an actual JSON response payload. We’ve saved that to a file in the application, test/support/weather_api_response.json. Let’s modify our test to use that data instead of the handwritten values we used earlier:

## Test Mocks

## Doctests?

## Running Tests in a Non-Mix Environment


## References
- [ExUnit Docs](https://hexdocs.pm/ex_unit/ExUnit.html)
- [Elixir School - Testing](https://elixirschool.com/en/lessons/testing/basics)
