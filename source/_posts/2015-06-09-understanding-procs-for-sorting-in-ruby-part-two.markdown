---
layout: post
title: "Understanding Procs for Sorting in Ruby – Part Two"
date: 2015-06-09 15:11:17 -0400
comments: true
categories: technical ruby
---

## Recap

[Last time](http://computerwalksintobar.com/blog/2015/06/08/understanding-procs-for-sorting-in-ruby-part-one/), we discussed procs and how to use them to set default blocks for Ruby methods.

A reminder:

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

<br><center>{% img http://media0.giphy.com/media/gtmEIOOOHVgHK/giphy.gif %}</center>

## Insertion Sort – Integers

According to [Wikipedia](http://en.wikipedia.org/wiki/Insertion_sort), insertion sort "is a simple sorting algorithm that builds the final sorted array (or list) one item at a time." Essentially, insertion sort works by keeping track of a sorted sub-list, and placing each element in the correct position in that sorted sub-list.

First, let's implement insertion sort with simple integers in ascending order.

So if we implement a method that inserts an element at the correct place in a sorted array, this sort should be easy. If the element is less than or equal to any other element in the sorted array, we should insert the new element before this existing element; otherwise, we should put the new element at the end:

```ruby
def insert_in_right_place(sorted_list, element)
  sorted_list.each_with_index do |list_el, i|
    return sorted_list.insert(i, element) if element <= list_el
  end
  sorted_list << element
end
```

Again, to sort an array of integers, we just have to iterate through it, placing each element in the sorted sub-list:

```ruby
def sort(array)
  sorted = []
  array.each do |element|
    sorted = insert_in_right_place(sorted, element)
  end
  sorted
end
```

This can be refactored as:

```ruby
def sort(array)
  array.inject([]) { |sorted, element| insert_in_right_place(sorted, element) }
end
```

## Sort By Condition

But what if we want to sort by an optional, user-entered block? ENTER: PROCS.

In Ruby, the spaceship operator `a <=> b` compares two items, returning 1 if `a > b`, 0 if `a == b`, and -1 if `a < b`. For example, `3 <=> 2` returns 1, whereas `2 <=> 3` returns -1. `2 <=> 2` returns 0.

Let's give our sort an optional block variable `&block`, so the user can enter his or her own sorting condition. The default condition should be the typical `a <=> b` comparison. Now, `#sort` becomes:

```ruby
def sort(array, &block)
  normalized_block = block || Proc.new{|a, b| a <=> b}
  array.inject([]) { |sorted, element| insert_in_right_place(sorted, element, normalized_block) }
end
```

This block becomes what we __check__ in `#insert_in_right_place`, which becomes:

```ruby
def insert_in_right_place(sorted_list, element, block)
  sorted_list.each_with_index do |list_el, i|
    return sorted_list.insert(i, element) if block.call(element, list_el) < 1
  end
  sorted_list << element
end
```

So let's try some examples:

```ruby
["aardvark", "deer", "elephant", "blowfish", "hamster", "crocodile", "cat"].sort
  => ["aardvark", "blowfish", "cat", "crocodile", "deer", "elephant", "hamster"] 
["deer", "blowfish", "hamster", "crocodile", "cat"].sort{ |a, b| a[1] <=> b[1] }
  => ["cat", "hamster", "deer", "blowfish", "crocodile"] 
```

<br><center>{% img http://media.giphy.com/media/JdCz7YXOZAURq/giphy.gif %}</center>