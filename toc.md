- What is Cuba?
- Installation
- Hola Cuba
- Using Cuba in a classic fashion
- Templates with Tilt
- Cuba's love affair with Mote
- Helpers
- Organizing your application
- Using middleware (cyx)
- Testing (the light and _easy_ way) (cyx)
- Shotgun (if you think it's slow, you're doing something wrong) (cyx)
- Deployment
- Authoring your own plugins
- Cuba Contrib

# What is Cuba?

Cuba is a microframework for web development originally inspired by
[Rum][rum], a tiny but powerful mapper for [Rack][rack] applications.

It integrates many templates via [Tilt][tilt], it's very easy to test
with either [rack-test][rack-test] or [Capybara][capybara].

# Installation

Cuba is a Ruby program and it's tested against Ruby 1.9.

It comes in gem format, and you can install it from the command line:

    $ gem install cuba

# Hola Cuba

Open up your text editor, and write the following file:

```ruby
require 'cuba'

Cuba.define do
  on root do
    res.write "Hello world"
  end
end

run Cuba
```

Save this file as `config.ru`. You can then try running this file:

```term
$ rackup config.ru
```

Going to http://localhost:9292/ should say the proverbial "Hello world".

## What's happening?

The fastest way to define a Cuba app is, as you might have guessed,
with the `Cuba.define do ... end` block. Within that block, you have
some objects available, such as `env`, `req` and `res`, and a very
useful function, `on`. Let's talk about `on` first.

### Flow control with `on`

If you know how to deal with nested `if`/`else` or nested `case` blocks,
you will get the idea behind `on` pretty easily.

It executes the passed block only if all of its arguments evaluate to
`true`. Let's look at this example:

```ruby
require 'cuba'

Cuba.define do
  on true, 2 + 2 == 4 do
    res.write "Hello world"
  end
end

run Cuba
```

As both arguments (`true` and `2 + 2 == 4`) evaluate to `true`, the
block is executed and the string `"Hello world"` is written to the
response object. You are not limited to having one `on` block, though.
Let's look at another example:

```ruby
require 'cuba'

Cuba.define do
  on false do
    res.write "This block will never execute"
  end

  on true do
    res.write "This block will execute"
  end
end

run Cuba
```

When one of the arguments to `on` evaluates to `false` or `nil`, the
block is skipped and the next call to `on` takes place. In this case,
only the last block will run. One last example will clarify what
happens when more than one call to `on` succeeds:

```ruby
require 'cuba'

Cuba.define do
  on true do
    res.write "This block will execute"
  end

  on true do
    res.write "This block will never execute"
  end
end

run Cuba
```

In this case, the first successful call to `on` wins. Once the
attached block is executed, the application will never evaluate
subsequent calls to `on`. This behavior is very important, because not
only it is key to Cuba's performance, but it can also be a huge gotcha
if you ignore it.

## Nested `on` blocks

What happens when you nest several `on` blocks is what you would
expect from nested `if`/`else` statements. Why don't we just use `if`
and `else`, you may ask: the answer is that `on` does a bit more, and
we will learn about that later.

```ruby
require 'cuba'

Cuba.define do
  on true do
    on true do
      on true do
        res.write "Hello from a deeply nested block"
      end
    end
  end
end

run Cuba
```

# Using Cuba in a classic fashion

# Templates with Tilt

Cuba ships with a plugin that provides helpers for rendering
templates. It uses [Tilt][tilt], a gem that interfaces with many
template engines.

``` ruby
require "cuba/render"

Cuba.plugin Cuba::Render

Cuba.define do
  on default do

    # Within the partial, you will have access to the local variable
    # `content`, that will hold the value "hello, world".
    res.write render("home.haml", content: "hello, world")
  end
end
```

Note that in order to use this plugin you need to have [Tilt][tilt]
installed, along with the templating engines you want to use.

Two more helpers are available, but let's first shed some light to the
configuration options.

The `Cuba::Render` plugin looks for settings in the `:render`
namespace:

```ruby
# The default path to views is `File.expand_path("views", Dir.pwd)`.
Cuba.settings[:render].store(:views, File.expand_path("tpl", Dir.pwd))

# The default template engine is `erb`.
Cuba.settings[:render].store(:template_engine, "haml")
```

# Cuba's love afair with Mote

# Helpers

# Organizing your application

# Using middleware

# Testing (the light and easy way)

# Shotgun (if you think it's slow, you are doing something wrong)

# Deployment

## Heroku

## Deploying on your own server

# Authoring your own plugins

Cuba comes with a very simple, yet powerful plugin system.

## Adding a couple of helper methods

## A more complete plugin

# Cuba Contrib

Over the course of the years, we've been able to identify the most basic
set of features that we've needed for the majority of our projects. We've
extracted most of these in a plugin, so others can benefit from our work.

## Getting cuba-contrib

## Setting up your project to use cuba-contrib

## Contributing to cuba-contrib.

[rum]: http://github.com/chneukirchen/rum
[rack]: http://github.com/chneukirchen/rack
[tilt]: http://github.com/rtomayko/tilt
[capybara]: http://github.com/jnicklas/capybara
[rack-test]: https://github.com/brynary/rack-test
