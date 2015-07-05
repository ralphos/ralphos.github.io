---
layout: post
title:  "A Quick Note on Rack"
date:   2015-07-05 11:00:00
categories: rack
published: true
---

If you've been developing Ruby on Rails applications for a while now you would have heard of "Rack" and "Rack middleware" many times. Here's a quick summary of what Rack is and what middleware does.

Taken from the [docs](http://www.rubydoc.info/github/rack/rack/master/file/README.rdoc):

> Rack provides a minimal, modular, and adaptable interface for developing web applications in Ruby. By wrapping HTTP requests and responses in the simplest way possible, it unifies and distills the API for web servers, web frameworks, and software in between (the so-called middleware) into a single method call.

__So what does this mean?__

Imagine you have a web server (WEBrick, Thin, Unicorn, Puma), Rack, and your Rails application. Rack will sit in between your web server and application and essentially wrap requests coming from your web server so they can be easily digested by your Rails application and returned using a simple interface.

> A Rack application is a Ruby object (not a class) that responds to call. It takes exactly one argument, the environment and returns an Array of exactly three values: The status, the headers, and the body.

A Rack application can be very simple - as described it is an object that responds to a method called `call`. Here are two quick examples of Rack applications that do exactly the same thing:

```ruby
# rack_app.ru

class RackApp
  def call(env)
    [200, "Content-Type" => "text/plain", ["Here's my response!"]]
  end
end

run RackApp.new
```

Or another way:

```ruby
# rack_app.ru

rack_app = Proc.new do |env|
  [200, "Content-Type" => "text/plain", ["Here's my response!"]]
end

run rack_app
```

The two examples above demonstrate how a Rack application is just a Ruby object, hence why a Proc can even be a Rack application.

You'll also notice the `call` method returns exactly three values: the **status**, **headers** and **body**. The status is represented by a status code (200, 404, 500) and indicates whether the request was successful or not. Headers are used to describe the resource being requested and the body will often be an HTTP page or JSON payload.

We can run the two applications above by ensuring we first have the rack gem installed:

`gem install rack`

and then at the command line, running:

`rackup rack_app.ru`

If you inspect the web server output, you'll see WEBrick running on port 9292:

```
[2015-07-05 11:48:46] INFO  WEBrick 1.3.1
[2015-07-05 11:48:46] INFO  ruby 2.0.0 (2013-06-27) [x86_64-darwin14.1.0]
[2015-07-05 11:48:46] INFO  WEBrick::HTTPServer#start: pid=5326 port=9292
```

And if we enter `12.0.0.1:9292` you should see in the browser:

> Here's my response!

Pretty straightforward. Part of the reason we're able to run the Rake application is becuase of the `.ru` extension. This signifies the file is a "rackup" file and enables us to use methods such as `run`. 

When thinking about Rack, the key takeaways to undestand are:

1. The context in which a Rack application is used (imagine it sitting between the web server and your Rails application);
2. That it is just an object, hence why it can be a class or Proc object;
3. It should have a method called `call` which returns exactly 3 values, the status, header & body.

### What's Rack middleware?

Rack middleware is a way to filter a request and response coming into your application. You can think of middleware as being similar to before/after action filters in your Rails' controllers. They enable you to do things like modify a request before it reaches your application or modify a response before it is returned to the browser.

An example of Rack middleware might be a simple logger:

```ruby
class RackApp
  def call(env)
    [200, "Content-Type" => "text/plain", ["Here's my response!"]]
  end
end

class Logger
  def initialize(app)
    @app = app
  end

  def call(env)
    puts "Calling " + env["PATH_INFO"]
    @app.call(env)
  end
end

use Logger

run RackApp.new
```

As you can see, the Rack middleware class follows some basic rules. For example, you must initialize your class taking in the application you want to wrap, as an argument. You must also define a `call` method which in turn calls the application's `call` method.

The idea is that your middleware is going to "decorate" your application and add some behaviour to the requests or responses.

Thanks to the Rack DSL, we don't need to intialize the middleware class as Rack takes care of this for us and we can simply put `use Logger` to tell Rack we want to use this piece of middleware.

When using this command, you can think of what's actually happening here as:

`run Logger.new(App.new)`

If you run the rack application you should see some output in your terminal with the basic logging appearing.

Rack middleware can be very powerful and reusable across Rack compatible applications. Hopefully the above gives you a quick insight into what Rack middleware is and how it applies to your application.
