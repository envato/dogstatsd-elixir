# dogstatsd-elixir

A fork of [adamkittelson/dogstatsd-elixir](https://github.com/adamkittelson/dogstatsd-elixir) with improvements and bug fixes.

A client for DogStatsD, an extension of the StatsD metric server for Datadog.

It is implemented as GenServer thus can be fitted into a supervision tree.

[![Build Status](https://travis-ci.org/envato/dogstatsd-elixir.svg?branch=master)](https://travis-ci.org/envato/dogstatsd-elixir)
[![Coverage Status](https://coveralls.io/repos/envato/dogstatsd-elixir/badge.png?branch=master)](https://coveralls.io/r/envato/dogstatsd-elixir?branch=master)

## Quick Start Guide

First install the library:

  1. Add dogstatsd to your `mix.exs` dependencies:

      ```elixir
      def deps do
        [
          {:dogstatsd, git: "https://github.com/envato/dogstatsd-elixir.git", tag: "0.0.4"}
        ]
      end
      ```

  2. Add `:dogstatsd` to your application dependencies:

      ```elixir
      def application do
        [applications: [:dogstatsd]]
      end
      ```
Then start instrumenting your code:

``` elixir
# Require the dogstatsd module.
require DogStatsd

# Configure DogStatsd.
{:ok, statsd} = DogStatsd.new("localhost", 8125)
```

Alternatively it can be placed in Application supervision tree:

```elixir
defmodule LoremIpsum.Application do
  use Application

  def start(_type, _args) do
    import Supervisor.Spec

    children = [
      worker(DogStatsd, [host: "localhost", port: 8125])
    ]

    opts = [strategy: :one_for_one, name: LoremIpsum.Supervisor]
    Supervisor.start_link(children, opts)
  end
end
```

Below are some examples how it is used:

```elixir
# Increment a counter.
DogStatsd.increment(statsd, "page.views")

# Record a gauge 50% of the time.
DogStatsd.gauge(statsd, "users.online", 123, %{sample_rate: 0.5})

# Sample a histogram
DogStatsd.histogram(statsd, "file.upload.size", 1234)

# Time a block of code
DogStatsd.time(statsd, "page.render") do
  render_page('home.html')
end

# Send several metrics at the same time
# All metrics will be buffered and sent in one packet when the block completes
DogStatsd.batch(statsd, fn(s) ->
  s.increment(statsd, "page.views")
  s.gauge(statsd, "users.online", 123)
end)

# Tag a metric.
DogStatsd.histogram(statsd, "query.time", 10, %{tags: ["version:1"]})
```

You can also post events to your stream. You can tag them, set priority and even aggregate them with other events.

Aggregation in the stream is made on hostname/event_type/source_type/aggregation_key.

``` elixir
# Post a simple message
DogStatsd.event(statsd, "There might be a storm tomorrow", "A friend warned me earlier.")

# Cry for help
DogStatsd.event(statsd, "SO MUCH SNOW", "Started yesterday and it won't stop !!", %{alert_type: "error", tags: ["urgent", "endoftheworld"]})
```

## Configuration

ENV | Use | Format | Default
--- | --- | --- | ---
DD_AGENT_HOST | The datadog-agent host | String | `127.0.0.1`
DD_DOGSTATSD_PORT| The datadog-agent Dogstatds port | String | `8125`

## Credits

* Based on [adamkittelson/dogstatsd-elixir](https://github.com/adamkittelson/dogstatsd-elixir)
* dogstatsd-elixir is a port of the [Ruby DogStatsd client](https://github.com/DataDog/dogstatsd-ruby)
