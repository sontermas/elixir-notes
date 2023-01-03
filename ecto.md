

Create a new project:
```
mix new linkly --sup
cd linkly
```

Here is the structure:
```
.
├── lib
│   ├── linkly
│   │   └── application.ex
│   └── linkly.ex
├── mix.exs
├── README.md
└── test
    ├── linkly_test.exs
    └── test_helper.exs
```

We need the `--sup` flag so that the project has a supervisor. This is for Ecto. Next, open the `mix.exs` file and add Ecto and Postgrex (a Postgres library) to your project's deps:
```elixir
defp deps do
  [
    {:postgrex, ">= 0.0.0"},
    {:ecto_sql, "~> 3.1"}
    # {:dep_from_hexpm, "~> 0.3.0"},
    # {:dep_from_git, git: "https://github.com/elixir-lang/my_dep.git", tag: "0.1.0"}
  ]
end
```

Fetch the deps:
```
mix deps.get
```

## Creating the repo
Create a repo file in the `lib/linkly` directory called, `repo.ex` and add the following contents to configure your Ecto repo:
```elixir
defmodule Linkly.Repo do
  use Ecto.Repo,
    otp_app: :linkly,
    adapter: Ecto.Adapters.Postgres
end
```

The repo also needs to be added into the supervision tree of the app, inside the start function in `application.ex`.
```
  def start(_type, _args) do
    children = [
      Linkly.Repo
```

## Configuration
Mix projects no longer generate configuration files by default, but we need some configuration in order to connect to a database. Create a `config` directory at the top level of your project and then a `config.exs` file in it to hold your default configuration and a `dev.exs` to hold the dev environment-specific configuration.

In `config.exs`, we'll add the ecto repos config and import the appropriate config for our environment:
```elixir
use Mix.Config

config :linkly, :ecto_repos, [Linkly.Repo]

import_config "#{Mix.env()}.exs"
```

We'll put off creating the `prod.exs` and `test.exs` config files for now and just create a `dev.exs` with our database credentials:
```elixir
use Mix.Config

config :linkly, Linkly.Repo,
  username: "postgres",
  password: "postgres",
  database: "linkly_dev",
  hostname: "localhost"
```

