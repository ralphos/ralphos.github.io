---
layout: post
title:  "Rails 4 with Unicorn & Foreman"
categories: Rails
published: true
---

As a first post I thought I'd start right from the beginning and give a quick overview of how I would set up a new project using Rails 4 with Ruby 2.0.

In this post we're going to run our app using [Unicorn](https://devcenter.heroku.com/articles/rails-unicorn) and [Foreman](https://github.com/ddollar/foreman) - which uses a `.env` file and `Procfile` to run processes just like Heroku's [Cedar](https://devcenter.heroku.com/articles/cedar/) stack.

#### Getting started

Make sure you've got Rails 4, Ruby 2.0 and PostgreSQL installed and then get started by running the following command

    $ rails new myapp -T -d postgresql

This command will create a new rails project and skip Test::Unit files (Rails default testing framework) as well as selecting PostgreSQL as the database (which we'll also be using on Heroku).

*Tip:* Use `$ rails new --help` as a good way to see what other options are available when starting a new project.

#### Running your app using Foreman & Unicorn

Foreman is an easy option to help you manage Procfile based applications. In
a nutshell, it let's you easily manage all your processes (background jobs etc)
in a single file, called a Procfile. [Unicorn](https://devcenter.heroku.com/articles/rails-unicorn) is a web server that lets you run multiple Ruby processes in a single dyno. In the following steps we're going to create a Procfile instructing Foreman to launch the Unicorn web server.

###### Unicorn setup

Start by adding Unicorn to your Gemfile

    gem 'unicorn'

Then run

    $ bundle install

Create a configuration file for Unicorn at config/unicorn.rb

    $ touch config/unicorn.rb

and copy the following into Unicorn.rb

{% highlight ruby %}
worker_processes Integer(ENV["WEB_CONCURRENCY"] || 3)
timeout 15
preload_app true

before_fork do |server, worker|
  Signal.trap 'TERM' do
    puts 'Unicorn master intercepting TERM and sending myself QUIT instead'
    Process.kill 'QUIT', Process.pid
  end

  defined?(ActiveRecord::Base) and
    ActiveRecord::Base.connection.disconnect!
end

after_fork do |server, worker|
  Signal.trap 'TERM' do
    puts 'Unicorn worker intercepting TERM and doing nothing. Wait for master to
send QUIT'
  end

  defined?(ActiveRecord::Base) and
    ActiveRecord::Base.establish_connection
end 
{% endhighlight %}

The above configuration will run 3 Unicorn
worker processes, which will enable your Rails app to support multiple concurrent
requests. (I suggest you [click here]
(https://devcenter.heroku.com/articles/rails-unicorn) if you want to understand
the specific Unicorn configurations above).


###### Create a Procfile for Foreman

Now that Unicorn is set up, we need to create a Procfile in the root directory of our application.

    # change directory into your project
    $ cd myapp

    # create the procfile by opening it up in your text editor
    $ vim Procfile

    # Note that the Procfile is case-sensitive and must be uppercase.

Edit your Procfile to tell Foreman to launch the application
using Unicorn.

    web: bundle exec unicorn -p $PORT -c ./config/unicorn.rb

If you haven't installed Foreman, run 

    $ gem install foreman 

NB: If you've installed the [Heroku Toolbelt](https://toolbelt.heroku.com/) you most likely have Foreman
already installed.

You can test whether your Procfile has been set up correctly by entering:

    $ foreman check

###### Create a .env file

Although not entirely necessary at this stage, it may be a good idea to create a
`.env` file. As per Heroku instructions you can set your `RACK_ENV` and `PORT`:

    $ echo "RACK_ENV=development" >>.env
    $ echo "PORT=5000" >> .env

Your `.env` should also be added to your `.gitignore` since this is specific to
your development environment.

    $ echo ".env" >> .gitignore
    $ git add .gitignore
    $ git commit -m "add .env to .gitignore"

I find the `.env` file to be super useful later on when managing environment
variables, especially when using Heroku since we're essentially using the same
set up.

#### Taking Foreman for a spin

OK we've followed all the steps. Everything should work BUT it won't. There are
still a couple of things you need to do.

Go to your `config/database.yml` and if the following line is present, remove it:

    $ username: myapp

This line most likely got added when we ran `$ rails new -d postgresql`. With this removed, Rails will now try to access the database as the user who is currently logged
into the computer.

Create your database

    $ rake db:create

And finally...

    $ foreman start

Your application should be running on [http://localhost:5000](http://localhost:5000)


You may want to commit these changes if you haven't done so already so we
can push the code to Heroku later.

    # initialize git repository
    $ git init

    # commit all files
    $ git add .

    # write your commit message
    $ git commit -m 'Initial commit with Unicorn & Foreman ready to use.'

> As a sidenote, you may also want to prepare your .gitignore file prior to your
initial commit. We already added our `.env` file and you may also want to do the
same for other files such as your `config/database.yml` or `vendor/` -
especially if you're using Rbenv like I am. I'll save the contents of my `.gitignore` for another post however.


That's it! Your application is up and running and almost ready to be deployed. With
this now set up you can either start writing code for your app or set up your
staging & production environments on Heroku which I'll be touching on in the
next post.
