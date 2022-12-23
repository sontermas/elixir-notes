# Testing

## ExUnit
Elixir's built-in test framework is `ExUnit` and it includes everything we need to test code. 

Tests are implemented as Elixir scripts so we need to use the `.exs` file extension. Before we can run our tests we need to start `ExUnit` with `ExUnit.start()`, this is most commonly done in `test/test_helper.exs`.

## Running Tests

### Using Mix
The test directory comprises two files. The file `test_helper.exs` will set the default configuration of all tests. This contains only one line of code `ExUnit.start` and the file `my_project_test.exs` is where our real test cases sit.

So basically, when you run the mix test task, it starts the current application, loads up test/test_helper.exs and then requires all files matching the `test/**/_test.exs` pattern in parallel. It loads all the files the have the suffix as `_test.exs`.

| Command | Description |
| ------- | ----------- |
| `mix test` | Run all the project’s tests. |
| `mix test test/file_test.exs` | Run all the file's tests. |
| `mix test test/file_test.exs:10` | Run the test case at line 10. |

### Not using Mix

## Assertions

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


## References
- [ExUnit Docs](https://hexdocs.pm/ex_unit/ExUnit.html)
- [Elixir School - Testing](https://elixirschool.com/en/lessons/testing/basics)
