---
layout: post
title: "How upgrade Node on Heroku Ruby Buildpack."
date: 2013-09-14 17:24
comments: true
categories: 
---

This was a real royal pain in the ass. I encountered a problem where
ruby-stylus no longer supported the version of Node that came with the default
ruby build pack. Steps below are really only for my own reference, if you're 
interested in something more detailed please let me know :)

You'll need an S3 account, `gem install vulcan`, and fork the [ruby
buildpack repo][buildpack].

Create a bucket on S3 for your binaries. **Make sure you use US Standard**.

Find the version of node that you want, `0.10.17` wasn't compiling
for me so I resorted to using `0.9.9` You can view a list [here][node-versions].

Run this to create the Heroku server that will be used to compile your 
binaries.

``` sh
vulcan create vulcan-YOURAPPNAME
```

Update `lib/language_pack/ruby.rb`, look for `NODE_VERSION` and put the
appropriate version in.

``` ruby lib/language_pack/ruby.rb
NODE_VERSION         = "0.9.9"
```

Update `lib/language_pack/base.rb`, look for `VENDOR_URL` and put your S3
bucket URL in.

``` ruby lib/language_pack/base.rb
VENDOR_URL = "https://s3.amazonaws.com/yourbucketname"
```

Open up `Rakefile`, update your `S3_BUCKET_NAME` to the bucket you just
created. Make sure that the line that downloads the node package looks 
like this:

``` ruby
desc "install node"
task "node:install", :version do |t, args|
  version = args[:version]
  name    = "node-#{version}"
  prefix  = "/app/vendor/node-v#{version}"
  Dir.mktmpdir("node-") do |tmpdir|
    Dir.chdir(tmpdir) do |dir|
      FileUtils.rm_rf("#{tmpdir}/*")

      sh "curl http://nodejs.org/dist/v#{version}/node-v#{version}.tar.gz -s -o - | tar vzxf -"

# ...
```

Newer distros of node are all under the URL format above Again [take a
look at this link][node-versions] to see if it applies to you.

Grant List and View permissions of your new bucket to everyone (Heroku will
need to use this bucket to download binaries.)

We need to make sure that we send the correct AWS credentials when uploading 
to the bucket:

``` sh
export S3_SECRET_ACCESS_KEY=youaccesskey
export S3_ACCESS_KEY_ID=accesskeyid
```

Finally we can compile node on the Vulcan Heroku App and store it on our
S3 account:

``` sh
rake node:install[9]
```

Since we are using our new amazon bucket to serve binaries, it will also
need to have copies of some of the other binaries:

``` sh
# Not sure if all of the below are needed
# Same versions specified in lib/language_pack/rubyrb
rake libffi:install[10]
rake libyaml:install[4]
rake ruby:install[0-p247] # Go get a coffee
rake gem:install[bundler,2]
```

To use your new build pack run this:

``` sh
heroku config:set BUILDPACK_URL=https://github.com/yourorg/yourbuildbpackrepo.git
```

Modifying the build pack seems interesting, there are probably a host of other
handy things you can do with it.

[buildpack]: https://github.com/heroku/heroku-buildpack-ruby
[node-versions]: http://nodejs.org/dist/
