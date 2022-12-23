# Testing

## References
- [ExUnit Docs](https://hexdocs.pm/ex_unit/ExUnit.html)
- [Elixir School - Testing](https://elixirschool.com/en/lessons/testing/basics)


Elixir's built-in test framework is `ExUnit` and it includes everything we need to test our code. 

Tests are implemented as Elixir scripts so we need to use the `.exs` file extension. Before we can run our tests we need to start `ExUnit` with `ExUnit.start()`, this is most commonly done in `test/test_helper.exs`.

We can run our project’s tests with `mix test`. 

| Macro | Description |
| -------- | ----------- |
| `assert` | Test that the expression is `true`.  |
| `refute` | Test that the expression is `false`. |
| `assert_raise`  | Assert that an error has been raised. |
| `assert_receive` | Test whether a message has been received. You can specify a timout. |
| `assert_received` | Test whether a message has been received. Will not wait for messages. |
| `capture_io` | Used to capture an application's output. |
| `capture_log` | Used to capture output to `Logger`. |
