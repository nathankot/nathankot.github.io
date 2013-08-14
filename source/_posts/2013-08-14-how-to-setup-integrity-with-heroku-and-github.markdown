---
layout: post
title: "How to setup Integrity with Heroku and Github."
date: 2013-08-14 22:29
comments: true
categories: guides
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


Now go to the Heroku app you just created and set up a project :)
