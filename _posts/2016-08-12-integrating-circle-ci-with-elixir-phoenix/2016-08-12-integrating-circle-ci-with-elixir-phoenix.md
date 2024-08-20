---
layout: post
title:  "Integrating CircleCI with Elixir/Phoenix"
date:   2016-08-12 09:29:20 +0100
tag: [devops, elixir]
image: "/integrating-circle-ci-with-elixir-phoenix/cicle-ci-phoenix.webp"
---

Having automated tests in a repository can be considered a _must-do_ in any kind of project. In Elixir/Phoenix we can create complex and maintainable tests in a short time but, how can we integrate it with CircleCI?

## Getting in touch with CircleCI

[CircleCI][circleci] is just another continuous integration platform, free for both public and private repositories.

The main advantage is that, as it’s based on top of containers, you can can access via SSH to the container where your tests are running and debug environment/build issues easily.

## So, what’s the matter with Elixir in CircleCI?

**CircleCI doesn’t have native support for Elixir/Phoenix projects**, so the only option left is trying to automatise the installation of the environment needed to run the tests.

This shouldn't be a real trouble since they allow you to define the shell commands executed at each step, so the process can be customised as much as you want.

In my case, I have decided to install the environment using [ASDF][asdf-vm], a new lightweight version manager that can handle multiple languages/platforms using plugins.

The idea is to install it (if it’s not previously installed) and apply the plugins needed. Then, we can use `.tool_versions` file (the default one used by ASDF) to install required dependencies and use them to run the actual tests.

So here is the circle.yml that does it:

```yaml
machine:
  environment:
    PATH: "$HOME/.asdf/bin:$HOME/.asdf/shims:$PATH"
    MIX_ENV: "test"
dependencies:
  cache_directories:
    - ~/.asdf
  pre:
    - if ! asdf | grep version; then git clone https://github.com/HashNuke/asdf.git ~/.asdf; fi
    - if ! asdf plugin-list | grep erlang; then asdf plugin-add erlang https://github.com/HashNuke/asdf-erlang.git; fi
    - if ! asdf plugin-list | grep elixir; then asdf plugin-add elixir https://github.com/HashNuke/asdf-elixir.git; fi
    - asdf plugin-update erlang
    - asdf plugin-update elixir
    - asdf install
    - cp config/test.ci.exs config/test.exs
    - yes | mix deps.get
    - yes | mix local.rebar
database:
  override:
    - mix ecto.drop
    - mix ecto.create
    - mix ecto.migrate
test:
  override:
    - mix test
experimental:
  notify:
    branches:
      only:
        - master
```

Also, some adjustments are made:

- Various environment variables, like the `$PATH` including ASDF paths
- Copying CircleCI specific project config to use it on tests
- Install required dependencies
- Clean the database
- Only notify the builds on master branch (to avoid flooding of notifications on _non-master_ commits).

Note that, by adding ASDF directory to the cache list, we can avoid reinstalling and rebuilding Erlang and Elixir, which will take (_really_) long time on first execution.

## Benchmarking

Thanks to the cache between builds, the build times of one of the projects we use at work are:

- ~3 minutes on regular builds.
- ~13 minutes on the first build (download and build Erlang & Elixir).


[circleci]: https://circleci.com
[asdf-vm]:  https://github.com/asdf-vm/asdf

