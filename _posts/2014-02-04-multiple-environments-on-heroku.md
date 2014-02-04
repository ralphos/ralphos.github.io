---
layout: post
title:  "Staging & Production Environments on Heroku"
date: 2014-02-04 10:59:56
categories: Rails Heroku
published: true
---

In this post I'll demonstrate a simple way to create staging and production environments on [Heroku](http://www.heroku.com) so you'll be slinging code on the web in no time.

Typically when working on an application, I'll create both staging and
production environments. This is a fairly common pattern in web development and
allows us to test new features in an environment very similar to our
live/production site. For the most part our staging environment will be very similar to our production environment, however we still need to set it up separately for added flexibility.

Carrying on from the previous post, let's create a staging configuration for our database in `config/database.yml` which should now look like this:

    development:
      adapter: postgresql
      encoding: unicode
      database: myapp_development
      pool: 5
      password:

    test:
      adapter: postgresql
      encoding: unicode
      database: myapp_test
      pool: 5
      password:

    staging:
      adapter: postgresql
      encoding: unicode
      database: myapp_staging
      pool: 5
      password:

    production:
      adapter: postgresql
      encoding: unicode
      database: myapp_production
      pool: 5
      password:

Before we forget we'll also create a `config/environments/staging.rb` by copying
the contents of `config/environments/production.rb` over. Simply change
settings/accounts for various services depending on your needs. *NB: Don't forget
to commit these changes!*

Now we can go ahead and create apps on Heroku with remote staging/production environments.

    $ heroku create myapp-staging --remote staging

    $ heroku create myapp-production --remote production

Then all we need to do is push to those git remotes:

    $ git push staging master

    $ git push production master

Migrate the databases

    $ heroku run rake db:migrate --remote staging

    $ heroku run rake db:migrate --remote production

Check that everything's working

    $ heroku ps --remote staging

    $ heroku ps --remote production

And that's it!

To manage staging and production configurations we can now set

    $ heroku config:set RACK_ENV=staging RAILS_ENV=staging --remote staging

and

    $ heroku config:set RACK_ENV=production RAILS_ENV=production --remote production

This will enable us to set different configuration variables for each environment. For example, you could add different S3 bucket credentials to your staging environment.

    $ heroku config:set S3_KEY=XXX --remote staging
    $ heroku config:set S3_SECRET=YYY --remote staging

For your local development environment you can simply put these environment
variables into your `.env` file (hence why we don't check this file into source
control).

    S3_key=xxxxxxxxxxxxxxxxxx
    S3_SECRET=xxxxxxxxxxxxxxxxx

That's all there is to creating staging & production environments on Heroku.
If you'd like to find out more about setting up such environments check out the
[Heroku docs](https://devcenter.heroku.com/articles/multiple-environments). 

Hopefully now you can see why I like this approach of using Foreman + Unicorn
along with a `.env` file to manage environment variables. I do believe it helps that
we employ a similar set up in both our develoment environment as well as staging
and production environments.
