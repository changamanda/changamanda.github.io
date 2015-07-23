---
layout: post
title: "Understanding Procs for Sorting in Ruby – Part One"
date: 2015-06-08 14:00:04 -0400
comments: true
categories: technical ruby
---

## What is a Proc, Anyway?

According to the Ruby documentation, "Proc objects are blocks of code that have been bound to a set of local variables. Once bound, the code may be called in different contexts and still access those variables." But what does that really _mean_?

<center>{% img http://media.giphy.com/media/zkSFsZpQMZuG4/giphy.gif %}</center>

Let's do some experimentation with Procs.

```ruby
proc = Proc.new{ |phrase| phrase.upcase }
  => #<Proc:0x007f99a20904c8@(irb):1> 

proc.call("hello!")
  => "HELLO!"
```

So it looks like a proc is a set of instructions inside of a variable. If I run the `#call` method on the proc, the arguments that I pass into `#call` will be passed into the block. Since I entered `proc.call("hello!")`, Ruby returned `"hello!".upcase`. Likewise, `proc.call("how's it going?")` returns `"HOW'S IT GOING?"`.

Let's try a longer one:

```ruby
capital_split = Proc.new do |phrase|
  phrase.upcase!
  phrase.split("")
end
  => #<Proc:0x007fb182859ce8@(irb):1>

capital_split.call("bingo")
  => ["B", "I", "N", "G", "O"]
```

Again, whatever we pass into `#call` gets passed into our block. This is starting to make sense!

## Normalizing Ruby Blocks

One important use of procs is their ability to set a *default block* for a Ruby method. Consider the method `#do_something` that takes a word and passes it into a block:

```ruby
def do_something(word)
  yield(word)
end

do_something("hello"){ |word| word.capitalize }
  => "Hello" 
```

But what if we want `#do_something` to capitalize its argument if if we don't call it with a block. As it stands, we'll get an error if we try to call `#do_something` without a block:

```ruby
do_something("hello")
LocalJumpError: no block given (yield)
```

If we pass in another argument that starts with `&`, our block gets captured in that variable, as a proc. So we can rewrite our `#do_something` method as:

```ruby
def do_something(word, &block)
  block.call(word)
end

do_something("hello"){ |word| word.capitalize }
  => "Hello"

do_something("hello")
NoMethodError: undefined method `call' for nil:NilClass
```

Now, setting a default block is a breeze:

```ruby
def do_something(word, &block)
  normalized_block = block || Proc.new{ |word| word.capitalize }
  normalized_block.call(word)
end

do_something("hello")
  => "Hello" 
do_something("hello"){ |word| word.capitalize }
  => "Hello" 
do_something("hello"){ |word| word.upcase }
  => "HELLO"
```

POWERFUL! Next time, we'll use our newfound proc powers to build a sort.

<center>{% img http://media.giphy.com/media/A9grgCQ0Dm012/giphy.gif %}</center>