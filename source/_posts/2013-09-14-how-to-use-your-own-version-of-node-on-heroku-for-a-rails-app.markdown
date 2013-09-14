---
layout: post
title: "How upgrade Node on Heroku Ruby Buildpack."
date: 2013-09-14 17:24
comments: true
categories: 
---

This was a real royal pain in the ass. I encountered a problem where
ruby-stylus no longer supported the version of Node that came with the default
ruby build pack.

Steps below are really only for my own reference, if you're interested in
something more detailed please let me know :)

You'll need an S3 account, `gem install vulcan`, and fork the [ruby
buildpack repo][buildpack].

Create a bucket on S3 for your binaries **Make sure you use US Standard**.

Find the version of node that you want, `0.10.17` wasn't compiling
for me so I resorted to using `0.9.9` You can view a list [here][node-versions].

Run `vulcan create vulcan-YOURAPPNAME`, this is the heroku server that will
be used to compile your binaries.

Update `lib/language_pack/rubyrb`, look for `NODE_VERSION` and put the
appropriate version in.

Update `lib/language_pack/baserb`, look for `VENDOR_URL` and put your S3
bucket URL in.

Open up `Rakefile`, update your `S3_BUCKET_NAME` to the bucket you just
created. Make sure that the line that downloads the node package looks 
like this:

``` ruby
sh "curl http://nodejsorg/dist/v#{version}/node-v#{version}targz -s -o | tar vzxf -"
```

Newer distros of node are all under the URL format above Again [take a
look at this link][node-versions] to see if it applies to you.

Grant List and View permissions of your new bucket to everyone (Heroku will
need to use this bucket to download binaries.)

We need to make sure that we send the correct AWS credentials when uploading 
to the bucket:

``` bash
export S3_SECRET_ACCESS_KEY=youaccesskey
export S3_ACCESS_KEY_ID=accesskeyid
```

Finally we can compile node on the Vulcan Heroku App and store it on our
S3 account:

``` bash
 rake node:install[9]
```

Since we are using our new amazon bucket to serve binaries, it will also
need to have copies of some of the other binaries:

``` bash
 # Not sure if all of the below are needed
 # Same versions specified in lib/language_pack/rubyrb
 rake libffi:install[10]
 rake libyaml:install[4]
 rake ruby:install[0-p247] # Go get a coffee
 rake gem:install[bundler,2]
```

Run `heroku config:set BUILDPACK_URL=https://githubcom/yourorg/yourbuildbpackrepo.git`
to use your new build pack!

Modifying the build pack seems interesting, there are probably a host of other
handy things you can do with it.

[buildpack]: https://github.com/heroku/heroku-buildpack-ruby
[node-versions]: http://nodejs.org/dist/
