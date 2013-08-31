---
layout: post
title: "How to setup Integrity with Rails, Heroku and Github."
date: 2013-08-14 22:29
comments: true
categories: guides
published: false
---

So I realized that [Ideasaurus](http://ideasaurus.io) really needs some CI. Did some
searching and found [Integrity](http://integrity.github.io/). Seems good, let's go!

First clone it:

    $ git clone https://github.com/integrity/integrity.git


Lets take a look at the `init.rb` config, it basically tells you what to do.
Just uncomment the Heroku-related stuff and comment out the conflicting stuff.

Make some changes to the `Gemfile`, you'll need to comment out the following gems
since Heroku can't build them:

    # gem 'dm-sqlite-adapter'
    # gem 'do_sqlite3'


Also make sure you uncomment the Postgres adapters:

    # Uncomment if you're using pg or mysql instead of sqlite
    gem 'pg'
    gem 'dm-postgres-adapter', '~> 1.2.0'


Bundle install:

    $ bundle install


Make a new heroku app:

    $ heroku apps:create
    $ heroku addons:add heroku-postgresql:dev


Setup a Github [post-receive webhook](https://help.github.com/articles/post-receive-hooks),
set the webhook url to: `http://YOURAPP.com/build/github/ANY_TOKEN_HERE` and set the `github_token`
to your `ANY_TOKEN_HERE`.

Choose a username/password you want to use with integrity:

    $ heroku config:set ADMIN_USERNAME=your_user_name ADMIN_PASSWORD=yourpassword

Deploy:

    $ git push heroku master --set-upstream
    $ heroku run rake db


Go to the Heroku app you just created and set up a project, you'll need to setup an
additional database on Heroku for your builds to use:

    $ heroku addons:add heroku-postgresql:dev # (Add another one)


Now copy and paste this script into `config/database.ci.yml`

    <%

    # This is taken from how Heroku does it.
    # Need to setup db this way for CI

    require 'cgi'
    require 'uri'

    begin
      uri = URI.parse(ENV["HEROKU_POSTGRESQL_PINK_URL"]) # <<< Replace this with your color
    rescue URI::InvalidURIError
      raise "Invalid DATABASE_URL"
    end

    raise "No RACK_ENV or RAILS_ENV found" unless ENV["RAILS_ENV"] || ENV["RACK_ENV"]

    def attribute(name, value, force_string = false)
      if value
        value_string =
          if force_string
            '"' + value + '"'
          else
            value
          end
        "#{name}: #{value_string}"
      else
        ""
      end
    end

    adapter = uri.scheme
    adapter = "postgresql" if adapter == "postgres"

    database = (uri.path || "").split("/")[1]

    username = uri.user
    password = uri.password

    host = uri.host
    port = uri.port

    params = CGI.parse(uri.query || "")

    %>

    <%= ENV["RAILS_ENV"] || ENV["RACK_ENV"] %>:
      <%= attribute "adapter",  adapter %>
      <%= attribute "database", database %>
      <%= attribute "username", username %>
      <%= attribute "password", password, true %>
      <%= attribute "host",     host %>
      <%= attribute "port",     port %>

    <% params.each do |key, value| %>
      <%= key %>: <%= value.first %>
    <% end %>


Replace the key marked above with the database url that you just generated.

Create a task with this:


    namespace :ci do

      task :copy_yml do
        system("cp #{Rails.root}/config/database.ci.yml #{Rails.root}/config/database.yml")
      end

      desc "Prepare for CI and run test suite."
      task :build => ['ci:copy_yml', 'db:test:prepare', 'spec'] do
      end

    end

In the build script of your project run this:

    bundle install
    rake ci:build

Happy building :)
