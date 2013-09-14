---
layout: post
title: "How to use your own version of Node on Heroku for a Rails app."
date: 2013-09-14 17:24
comments: true
categories: 
---

This was a real royal pain in the ass. I encountered a problem where
ruby-stylus no longer supported the version of Node that came with the default
ruby build pack.

Steps below are really only for my own reference, if you're interested in
something more detailed please let me know :)

1. You'll need an S3 account, `gem install vulcan`, and fork the [ruby
   buildpack repo][buildpack].

2. Create a bucket on S3 for your binaries.

3. Find the version of node that you want, `0.10.17` wasn't compiling
   for me so I resorted to using `0.9.9`. You can view a list [here][node-versions].

4. run `vulcan create vulcan-YOURAPPNAME`, this is the heroku server that will
   be used to compile your binaries.

5. Update `lib/language_pack/ruby.rb`, look for `NODE_VERSION` and put the
   appropriate version in.

6. Update `lib/language_pack/base.rb`, look for `VENDOR_URL` and put your S3
   bucket url in.

7. Open up `Rakefile`, update your `S3_BUCKET_NAME` to the bucket you just
   created.  Make sure that the line that downloads the node package looks 
   like this:

   ``` ruby
   sh "curl http://nodejs.org/dist/v#{version}/node-v#{version}.tar.gz -s -o - | tar vzxf -"
   ```

   Newer distros of node are all under the URL format above. Again [take a
   look at this link][node-versions] to see if it applies to you.

8. Grant List and View permissions to your new bucket to everyone (Heroku will
   need to use this bucket to download binaries.)

9. We need to make sure that the vulcan heroku app has the correct AWS 
   credentials and bucket name, run this:

   ``` bash

   ```
10. Finally we can compile node on the vulcan heroku app and store it on your
   s3 account:

   ``` bash
   ```

11. Since we are using our new amazon bucket to serve binaries, it will also
   need to have copies of _all the other binaries_:

   ``` bash
   ```

12. Run `heroku config:set BUILDPACK_URL=https://github.com/yourorg/yourbuildbpackrepo.git`
    to use your new build pack!


[buildpack]: https://github.com/heroku/heroku-buildpack-ruby
[node-versions]: http://nodejs.org/dist/
