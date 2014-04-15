---
layout: post
title:  "ActiveAdmin using Clearance on Rails 4.1"
date: 2014-04-15 10:50:21
categories: Rails
published: true
---

Recently I worked on a project that required an admin interface for the
business owner to manage records within the application. To this end we
considered a few different approaches.

* [ActiveAdmin](http://activeadmin.info/)
* [RailsAdmin](https://github.com/sferik/rails_admin)
* Custom

Since the goal for this project was to build an MVP (minimum viable product) it
was agreed building a custom solution would be too much work. After spending
time reviewing both ActiveAdmin and RailsAdmin, a decision was made to go ahead
with ActiveAdmin. We much preferred the ActiveAdmin interface and didn't like the idea of applying configuration in our models for RailsAdmin.

Notwithstanding, ActiveAdmin does have a few of it's own drawbacks. The first of
which is it's dependency on using [Devise](https://github.com/plataformatec/devise) for authentication, which conflicts
with our preference for using [Clearance](https://github.com/thoughtbot/clearance).

Hopefully this post saves you a bit of time as I show you how we set up ActiveAdmin to use Clearance in a Rails 4.1 application.

In our application we have an existing `User` model which uses Clearance for authentication. Therefore we just want to add ActiveAdmin without using Devise.

Rails 4 support for ActiveAdmin is still in progress so we have to add the following to our `Gemfile`.

{% highlight ruby %}
gem 'activeadmin', github: 'gregbell/active_admin'
gem 'polyamorous', github: 'activerecord-hackery/polyamorous'
gem 'ransack', github: 'activerecord-hackery/ransack', branch: 'rails-4.1'
gem 'formtastic', github: 'justinfrench/formtastic'
{% endhighlight %}

Install the gems

    $ bundle install

And then run the generator to use the existing users table

    $ rails g active_admin:install --skip-users

Migrate the database

    $ bundle exec rake db:migrate

Add assets for precompilation when deploying to Heroku (in `config/environments/production.rb`)


{% highlight ruby %}
config.assets.precompile += ['active_admin.js',
                             'active_admin.css',
                             'active_admin/print.css']
{% endhighlight %}

Then turn on serving static assets in `production.rb` as well


{% highlight ruby %}
config.serve_static_assets = true
{% endhighlight %}

Now for the bit which seems a bit tricky but actually is quite straightforward.

In `config/initializers/active_admin.rb` we want to override some of the existing default methods to tell ActiveAdmin to use Clearance as our authentication solution. We can do this by adding:

{% highlight ruby %}
config.authentication_method = :authenticate_admin_user!
{% endhighlight %}

And then in `application_controller.rb` we can define the `:authenticate_admin_user!` method to be

{% highlight ruby %}
def authenticate_admin_user!
  redirect_to sign_in_path unless current_user.try(:admin?)
end
{% endhighlight %}

As it says, this method will simply redirect to Clearance's `sign_in_path` unless the user is an admin, which is defined in our `User.rb`

{% highlight ruby %}
def admin?
  self.email && ENV['ADMIN_EMAILS'].to_s.include?(self.email)
end
{% endhighlight %}

To check if you're an admin we first see if you have an email address and then we see if that email address is included in our ADMIN_EMAIL environment variable, which can be found in a `.env` file.

`ADMIN_EMAILS=email@example.com,another_email@example.com`

If our user is an admin, we can redirect them to the admin dashboard after they sign in by adding the following to our `sessions_controller.rb`


{% highlight ruby %}
# ...

protected

def url_after_create
  if current_user.admin?
    redirect_to admin_dashboard_url
  end
end
{% endhighlight %}

Or we can just add a link in the nav bar

{% highlight erb %}
<% if current_user.admin? %>
  <%= link_to 'Admin Dashboard', admin_dashboard_path %>
<% end %>
{% endhighlight %}

Additionally, you will want to update the following settings in `config/intializers/active_admin.rb`.

Change

{% highlight erb %}
config.current_user_method = :current_admin_user
{% endhighlight %}

to

{% highlight erb %}
config.current_user_method = :current_user
{% endhighlight %}

and

{% highlight erb %}
config.logout_link_path = :destroy_admin_user_session_path
{% endhighlight %}

to

{% highlight erb %}
config.logout_link_path = :sign_out_path
{% endhighlight %}

and

{% highlight erb %}
config.logout_link_method = :get
{% endhighlight %}

to

{% highlight erb %}
config.logout_link_method = :delete
{% endhighlight %}

All we're doing above is changing the ActiveAdmin settings to use the helper methods set by Clearance.

Now that we have ActiveAdmin set up to use Clearance, we can start generating resources for ActiveAdmin. For example:

`$ rails g active_admin:resource User`

Fire up your rails server (or [Foreman](https://github.com/ddollar/foreman) if you're like me) and assuming you login with an email address included in your ADMIN_EMAILS variable, you should see the admin dashboard page generated by ActiveAdmin!
