---
layout: post
title:  "How I Start with TDD using RSpec & Capybara"
date:   2014-02-13 11:00:00
categories: Rspec Capybara
published: true
---

You've got your Rails application up and running, you may have even deployed it to
Heroku - and now you want to start coding using TDD (test-driven
development), but you're asking yourself - where do I start?

In general, I try to adhere to an outside-in approach to development whereby I'll start
with an integration spec (also known as a 'feature spec'), which will tell me
what to do and ultimately lead me to unit testing.

We use the phrase outside-in development since we'll be starting from the outside of the
application - in other words at the user interface layer - where our specs will
be driven by the actions a user takes on the site. We'll then drop down to the
inner layer of our application, to unit test individual methods in our Models.

We'll use RSpec as our testing framework over Test::Unit because the syntax
encourages human readable tests.

We'll also use a Ruby gem called Capybara in conjunction with Rspec, to include steps in our specs such as:


{% highlight ruby %}
visit root_path
fill_in 'Login', :with => 'user@example.com'
fill_in 'Password', :with => 'password'
click_link 'Sign in'
{% endhighlight %}

Thanks to the Capybara DSL (domain-specific language) demonstrated above, it's easier to simulate how a real user would use our application, to the extent where we may not even have to open a browser to click-test every new feature.

So let's begin by adding Rspec & Capybara to our `Gemfile`

{% highlight ruby %}
group :development, :test do
  gem 'rspec-rails'
end

group :test do
  gem 'capybara'
end
{% endhighlight %}

Next, install the gems via Bundle command

    $ bundle install

and install RSpec using the custom generator

    $ rails g rspec:install

which creates two files and a `spec/` directory:

    .rspec
    create  spec
    create  spec/spec_helper.rb

Now that you have RSpec installed, let's finish setting up Capybara by visiting
`spec/spec_helper.rb` and adding:

{% highlight ruby %}
require 'capybara/rspec'
{% endhighlight %}

We'll also create a `features/` folder to hold all our feature specs

    $ mkdir spec/features

and finally create our first feature spec

    $ touch spec/features/visitor_sees_homepage_spec.rb

As you can see from the name of the spec file, I'm trying to follow a pattern of
naming my spec files based around an actor and the action they are
undertaking.

Inside this feature spec I'll write my very first failing spec:

{% highlight ruby %}
require 'spec_helper'
{% endhighlight %}

and running it with the following command:

    $ bundle exec rspec spec/features/visitor_sees_homepage_spec.rb

This may seem trivial but it's an important step in adopting a holistic TDD
approach.

> Tip: You should aim to do the least amount of work possible and let the
tests drive your next move.

Our first spec will eventually look like this.

{% highlight ruby %}
require 'spec_helper'

feature 'Visitor sees homepage' do
  scenario 'with welcome text' do
    visit root_path

    expect(page).to have_content 'Welcome to my Homepage'
  end
end
{% endhighlight %}

In the above example, where we're simply telling our spec to visit the
`root_path` (our homepage) and check whether the content 'Welcome to my
Homepage' exists.

We'll continue to write more feature specs like the one above (with more
detail of course) and begin writing unit tests when our feature specs begin to fail due to
a method missing in our model, for example.

Hopefully this gives you an idea as regards to applying a TDD approach when
building your next application!

P.S. You may also want to consider adding other gems to your test environment such as [Factory
Girl] [1] for test data, [Shoulda Matchers][2] for common RSpec matchers, [Timecop][3] for testing time, [Launchy][4] to use Capybara's `save_and_open_page` commanda, and later on [Capybara-webkit][5] for headless browser testing and [Database Cleaner][6] to ensure a clean state for testing.


[1]: https://github.com/thoughtbot/factory_girl_rails 'Factory Girl Rails'
[2]: https://github.com/thoughtbot/shoulda-matchers 'Shoulda Matchers'
[3]: https://github.com/travisjeffery/timecop 'Timecop'
[4]: https://github.com/copiousfreetime/launchy 'Launchy'
[5]: https://github.com/thoughtbot/capybara-webkit 'Capybara-webkit'
[6]: https://github.com/bmabey/database_cleaner 'Database Cleaner'
