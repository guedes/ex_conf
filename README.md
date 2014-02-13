# ExConf
> Simple Elixir Configuration Management



## Features
- Configuration definitions are *evaluated at runtime*, but merged/defaulted at compile time, allowing runtime dependent lookups. (i.e. System.get_env)
- Configuration modules can extend other configurations for overrides and defaults
- Evironment based lookup for settings based on current Mix.env

## Simple Example
```elixir

defmodule MyApp.Config do
  use ExConf.Config

  config :router, ssl: true, 
                  domain: "example.dev",
                  port: System.get_env("PORT")
                  
  config :session, secret: "secret"
end

iex> MyApp.Config.router[:domain]
"example.dev"


defmodule MyApp.OtherConfig do
  use MyApp.Config

  config :session, secret: "123password"
end

iex> MyApp.OtherConfig.session[:secret]
"123password"
iex> MyApp.OtherConfig.router[:ssl]
true
```


## Environment Based Configuration

First, establish a *base* configuration module that uses `ExConf.Config`.
```elixir
defmodule MyApp.Config do
  use ExConf.Config

  config :router, ssl: true
  config :twitter, api_token: System.get_env("API_TOKEN")
end
```

Next, define "submodules" for each Mix.env you need overrides or additional settings for.
The *base* config module will look for a "submodule" whos name is `Mix.env` capitalized.
This allows environment specific lookup at runtime via the `env/0` function on the base module.
If the Mix.env specific config module does not exist, it falls back to the base module.

Here's what a Dev enviroment config module would look like for `Mix.env == :dev`:
```elixir
defmodule MyApp.Config.Dev do
  use MyApp.Config

  config :router, ssl: false
  config :twitter, api_token: "ABC"
end

iex> Mix.env
:dev

iex> MyApp.Config.env
MyApp.Config.Dev

iex> MyApp.Config.env.router[:ssl]
false
```

