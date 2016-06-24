---
layout: post
title: "Monad's and cookie jars"
date: 2016-06-23 22:22:11 -0700
comments: true
categories: 
---

_I've come across a handful of analogies for monads. Unfortunately I don't
remember I lot of them being very good at explaining why monad's are
useful. Here's my attempt._

Imagine you're at the end of a long line of people. There's a stack of cookies
that gets passed down from the start of the line. You're eager to try one of
these cookies but you have no idea when it'll be your turn, or even if there are
enough cookies to go around. The problem here is that given a line of unknown
length, you can never be sure whether the stack of cookies have already been
depleted or that it's just taking a while to reach you.

Enter the cookie jar. Now if you know that these cookies are stored in a jar,
and that even if the jar is empty it will still be passed down the line. You can
now know for sure that if you have not received the cookie jar, it is simply
taking a long time to reach you.

The cookie jar represents what is called a `Maybe` monad. It either does or
doesn't have cookies, regardless you'll still get the jar and pass it on.
