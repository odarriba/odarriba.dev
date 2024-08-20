---
layout: post
title:  "Sending notifications in Elixir with Ravenx"
date:   2017-04-25 17:45:20 +0100
tag: [notifications, libraries, elixir]
image: "/sending-notifications-in-elixir-with-ravenx/image.webp"
---

During the development of the new version of [Acutario][acutario] in Elixir, we had some issues related to notification dispatching.

One of the improvements we are working on is the possibility of sending notifications to multiple services, like e-mail, Slack, push notifications to mobile apps…

That functionality make us face with the problem of using different strategies depending on a concrete scenario that can be different for each user we want to notify, so we decided to develop a library that help us to manage all that cases. **And we called it [Ravenx][ravenx].**

## How Ravenx works

The library is made of **strategies**, which define a way of sending a notification (Slack, e-mail, APNS, etc), and they have an [standard interface][standard-interface], so its internal behaviour is abstracted from the outside.

Currently, Ravenx has two different strategies:

- **Slack**: provides sending slack messages using webhooks URLs.
- **E-mail**: provides sending emails using different strategies (the ones included in [bamboo][bamboo]).

But, as we are going to see at the end of this article, new strategies can be easily implemented to manage sending notifications to other systems and platforms.

Also, there are several ways of defining **configurations** which the strategies will use to know how to send the notification.

Configurations are per strategy, so they defines how a notification is sent (for example, Mailgun’s configuration to send e-mails).

## Global configuration

This approach includes two kinds of configuration:

- **General Ravenx configuration:** like the custom strategies to use, or the configuration module of our app (we will see this two aspects in a few moments).
- **Per-strategy configuration:** _strategy-specific_ configuration (Mailgun’s config, Slack’s webhook URL…)

```elixir
config :ravenx, :slack,
  url: "...",
  icon: ":bird:"
```

## Configuration module

A module that have a function for each strategy that receives the payload and returns a map with configurations that apply to that specific payload.

These configuration modules are useful to automatise the configuration generation, for example, if we need to retrieve the slack username of each user in each notification.

```elixir
config :ravenx,
  config: YourApp.RavenxConfig
```
&nbsp;
```elixir
defmodule YourApp.RavenxConfig do
  def slack (_payload) do
    %{
      url: "...",
      icon: ":bird:"
    }
  end
end
```

## Particular configuration
This is a configuration map sent when a notification is dispatched. Useful to add configuration generated right before sending the notification.

```elixir
Ravenx.dispatch(strategy, payload, %{url: "...", icon: ":bird:"})
```

## Mixing configurations
For a given notification dispatch we can have configurations of the three kinds. In that case, configurations are merged with this priorities:

> Global < Module < Particular

So a particular configuration key is used even if it’s defined in the configuration returned by a configuration module or by the app configuration.

## Sending a notification

**Ravenx** allows to send a notification in a **synchronous or asynchronous** way. In case you want to send a notification in an asynchronous way, a Task will be launched to perform the operation under the hood.

To send a notification, the only action needed is a call to the library:

```elixir
Ravenx.dispatch(:slack, %{title: "Hello world!", body: "Science is cool"})
# => {:ok, "ok"}

Ravenx.dispatch(:wadus, %{title: "Hello world!", body: "Science is cool"})
# => {:error, {:unknown_strategy, :wadus}}

{status, task} = Ravenx.dispatch_async(:slack, %{title: "Hello world!", body: "Science is cool"})
# => {:ok, %Task{owner: #PID<0.165.0>, pid: #PID<0.183.0>, ref: #Reference<0.0.4.418>}}

Task.await(task)
# => {:ok, "ok"}

Ravenx.dispatch_async(:wadus, %{title: "Hello world!", body: "Science is cool"})
# => {:error, {:unknown_strategy, :wadus}}
```

## Sending multiple notifications

Sending a standalone notification is cool, but no one needs another library for that. **The true functionality in Ravenx is sending a notification using multiple strategies in an easy way.**

To do that, the library defines a macro that can be used to generate **notification modules**, which can be used to define how to send a particular notification using multiple strategies.

That modules must define a function called `get_notifications_config` that receives an object and returns a Keyword list with information of which strategies and which payloads will be used:

```elixir
YourApp.Notification.NotifyUser.dispatch(user)
# => [
#   slack: {:ok, ...},
#   email_user: {:ok, ...},
#   email_company: {:ok, ...},
#   other_notification: {:error, {:unknown_strategy, :invalid_strategy}}
# ]

YourApp.Notification.NotifyUser.dispatch_async(user)
# => [
#   slack: {:ok, %Task{..}},
#   email_user: {:ok, %Task{..}},
#   email_company: {:ok, %Task{..}},
#   other_notification: {:error, {:unknown_strategy, :invalid_strategy}}
# ]
```
&nbsp;
```elixir
defmodule YourApp.Notification.NotifyUser do
  use Ravenx.Notification

  def get_notifications_config(user) do
    # In this function you can generate your payloads and decid which strategies you
    # want to use depending on the obejct received, and return something like:
    [
      slack: {:slack, %{title: "Important notification!", body: "Wait..."}, %{channel: user.slack_username}},
      email_user: {:email, %{subject: "Important notification!", html_body: "<h1>Wait...</h1>", to: user.email_address}},
      email_company: {:email, %{subject: "Important notification about an user!", html_body: "<h1>Wait...</h1>", to: user.company.email_address}},
      other_notification: {:invalid_strategy, %{text: "Important notification!"}, %{option1: value2}},
    ]
  end
end
```

So that allows to define all the logic behind a notification generation in a particular module.

That modules can be used (as seen above) to dispatch the notifications in both synchronous and asynchronous way, as when we dispatched simple notifications.

## Custom strategies

Anyone can create custom strategies and integrate them with Ravenx. To do that, the only requirement is to implement the interface needed and tell Ravenx which modules should use as strategies:

```elixir
config :ravenx,
  strategies: [
    my_strategy: YourApp.MyStrategy
  ]
```
&nbsp;
```elixir
defmodule YourApp.MyStrategy do
  @behaviour Ravenx.StrategyBehaviour

  def call(payload, options) do
    # ... do stuff ...
  end
end
```

## What Ravenx doesn’t do

**Ravenx doesn’t take care of dispatching schedule** (limiting the amount of notifications sent simultaneously). As Erlang VM can virtually take care of a lot (_really, a lot_) of internal processes, this is not a real issue.

You can send a lot of notifications asynchronously and the Erlang VM will be able to manage them without much problem.

## Wrapping up

Ravenx is not the perfect solution for all the notification scenarios (and it is not intended to be that), but in [Acutario][acutario] we think this can solve some complex scenarios in which you can have to deal with changing configurations, multiple strategies and massive dispatches of notifications.

Using notification modules alongside with the configuration options, the code related to notifications can be organised and reused, avoiding repetitions in multiple places of your code.

**If you have any question about Ravenx**, you can read its [documentation][docs] here, ask any question here or just [open an issue][ravenx]! (Pull requests are welcome too :)

Hope you enjoyed this article and see you soon (in ElixirConf.EU if you assist!)


[acutario]: http://www.acutar.io/
[ravenx]: https://github.com/acutario/ravenx
[standard-interface]: https://github.com/acutario/ravenx/blob/master/lib/ravenx/strategy_behaviour.ex
[bamboo]: https://github.com/thoughtbot/bamboo
[docs]: https://hexdocs.pm/ravenx/
