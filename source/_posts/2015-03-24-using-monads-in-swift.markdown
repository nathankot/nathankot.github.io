---
layout: post
title: "Using monads in Swift"
date: 2015-03-24 16:18:52 +0900
comments: true
categories: swift monad
---

One of the coolest features of Swift is that it lets you define your own
operators. This leads to being able to re-implement many operators found in more
functional languages that we know and love.

[Swiftz][swiftz] is a new[^1] library that implements a handful of essential and obscure
operators, as well as many structs that we take for granted in functional
environments. Today I'm going to take a brief look at the `Maybe` monad provided
by Swiftz.

Let's implement a game of chinese whispers. But in this version, any participant
can give up on the game. We have the participants, each a function that _might_
return a phrase that they think is correct, or nothing at all:

```swift
func participant1(phrase: String) -> Maybe<String> {
  return Maybe.just(phrase)
}

func participant2(phrase: String) -> Maybe<String> {
  return Maybe.just("aloha")
}

func participant3(phrase: String) -> Maybe<String> {
  return Maybe.just("\(phrase) i think")
}

func participant4(phrase: String) -> Maybe<String> {
  return Maybe.none()
}
```

Now how do we write a function that takes a phrase and an ordered list of
participants, and then returns the result? We could do this:

```swift
func play(initialPhrase: String, participants: [String -> Maybe<String>]) {
  var result: Maybe<String> = Maybe.just(initialPhrase)
  for participant in participants {
    if result.isNone() { break }
    result = participant(result.fromJust())
  }
  return result
}
```

You'll notice that we are checking for `isNone()` on each iteration because we
don't want to pass the next participant anything if the game has already
stopped. This is the problem that Monad's solve, given the context of `Maybe` it
_knows_ how to handle the case of nothing or something:

```swift
func play(initialPhrase: String, participants: [String -> Maybe<String>]) {
  var result: Maybe<String> = Maybe.just(initialPhrase)
  for participant in participants {
    // >>- is the equivalent of >>= from haskell, it's
    // called the `bind` operator
    result = result >>- participant
  }
  return result
}
```

Okay. That saved about one line, but we can make the entire thing more
functional:

```swift
func play(initialPhrase: String, _ transform: String -> Maybe<String>) {
  return transform(initialPhrase)
}

// And usage looks like:
play("hello there", { phrase: String -> Maybe<String> in
  return Maybe.just(phrase) >>- participant1 >>- participant2 >>- participant3
})

// Or, IF Swift supported partial operator application:
play("hello there, (>>- participant1 >>- participant2 >>- participant3) • Maybe.just)
// • is an operator defined by Swiftz (compose)
```

[^1]: Swiftz is not well documented yet, but the source code is clean enough to understand what's going on.

[swiftz]: https://github.com/typelift/Swiftz
