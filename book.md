# Cuba Fundamentals

Cuba is a microframework for web development originally inspired
by [Rum][rum], a tiny but powerful mapper for [Rack][rack]
applications.

Install the gem with the following command:

    $ gem install cuba

You can also read more about it at [cuba.is](http://cuba.is).


## Hola Cuba

Open up your text editor, and write the following lines:

```ruby
require 'cuba'

Cuba.define do
  on root do
    res.write "Hello world"
  end
end

run Cuba
```

Save this file as `config.ru`. You can now try running it:

```term
$ rackup config.ru
```

Going to http://localhost:9292/ should say the proverbial "Hello
world".

The fastest way to define a Cuba app is, as you might have
guessed, with the `Cuba.define do ... end` block. Within that
block, you have some objects available, such as `env`, `req` and
`res`, and a very useful function, `on`. Let's talk about `on`
first.


## Flow control with `on`

If you know how to deal with nested `if`/`else` or nested `case`
blocks, you will get the idea behind `on` pretty easily.

It executes the passed block only if all of its arguments evaluate
to `true`. Let's look at this example:

```ruby
require 'cuba'

Cuba.define do
  on true, 2 + 2 == 4 do
    res.write "Hello world"
  end
end

run Cuba
```

As both arguments (`true` and `2 + 2 == 4`) evaluate to `true`,
the block is executed and the string `"Hello world"` is written to
the response object. You are not limited to having one `on` block,
though. Let's look at another example:

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

When one of the arguments to `on` evaluates to `false` or `nil`,
the block is skipped and the next call to `on` takes place. In
this case, only the last block will run. One last example will
clarify what happens when more than one call to `on` succeeds:

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
subsequent calls to `on`. This behavior is very important, because
not only it is key to Cuba's performance, but it can also be a
huge gotcha if you ignore it.


### Nested `on` blocks

What happens when you nest several `on` blocks is what you would
expect from nested `if`/`else` statements. Why don't we just use
`if` and `else`, you may ask: the answer is that `on` does a bit
more, and we will learn about that later.

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

All the blocks will get executed, and the response will contain
the passed string.

One last important detail about how `on` blocks work can be
illustrated in the following example:

```ruby
require 'cuba'

Cuba.define do
  on true do
    on true do
      res.write "Hello from a deeply nested block"
    end

    res.write "This won't be evaluated"
  end
end

run Cuba
```

As `on` blocks halt and exit immediately, everything that comes
after a matching block, even if it looks like it should get
evaluated, will be skipped. Internally, Cuba uses `throw`/`catch`,
which explains that behavior.


## Accessing the environment

### Using `env`

The `env` object contains all the information of the request in
its raw form. All the CGI environment variables are present and
accessible:

```ruby
# Display the HTTP verb
res.write env["HTTP_REQUEST_METHOD"]
```

You can inspect the contents of the `env` variable directly, but
usually it's easier to access its values with the `req` object,
which is just a bit more than a wrapper with some syntax sugar.

### Using `req`

The `req` objects provides helpers and shortcuts for accessing the
request environment variables. For example, if you want to display
the request method as in the previous section, you can instead
call `req.request_method`. Alternatively, you can ask whether a
request was a POST with the `req.post?` helper. Here's a short
list of some methods provided by the `req` object:

```ruby
# Inspecting the request method
req.delete?
req.get?
req.head?
req.options?
req.patch?
req.post?
req.put?
req.trace?

# XMLHttpRequest verification
req.xhr?

# Accessing the query string
req.query_string

# Accessing the passed parameters
req.params

# Accessing the user agent
req.user_agent
```

You can read the documentation for `Rack::Request` for more
information about the methods available and how to use them.

### Using `res`

In the previous examples, we have used `res` to write strings
that are returned to the browser. It is an instance of
`Cuba::Response`, an object that defines methods for setting the
response status, the headers and the body.

#### Default values

The default status of the `res` instance is 200, but if no routes
match the request it is changed to 404 to denote that no resource
was found. You can override its value:

```ruby
# Setting a different status
res.status = 206
```

The default hash of headers is:

```ruby
{ "Content-Type" => "text/html; charset=utf-8" })
```

You can add, modify or remove headers by accessing the hash:

```ruby
res.headers["Content-Type"] = "application/json"
```

The default body is an empty array. When you write to the `res`
object, the strings are appended to `res.body` and a header with
the length of the response is calculated. Multiple calls to
`res.write` are possible.

```ruby
res.write "Hello, "
res.write "world"
```

#### Redirecting

The `res` object also provides a helper for HTTP redirection.

```ruby
res.redirect "http://example.com"
```

It sets the passed value to the "Location" header and updates the
status to 302. An optional second parameter corresponds to the
status code.

#### Cookies

You can add or remove cookies with these two methods:

```ruby
res.set_cookie "some-key", "some-value"
res.delete_cookie "some-key"
```

#### Rack API

The Rack API expects an array with exactly three elements: a
status code, a hash of headers and an enumerable body. Cuba
generates this array by calling `res.finish`. If you want to check
how the response looks like at any point in time, you can print
the result of `res.finish`.

```ruby
# Render the current value of res.finish
res.write res.finish.inspect
```

[rum]: http://github.com/chneukirchen/rum
[rack]: http://github.com/chneukirchen/rack
