---
layout: post
title: "Bunnies && Generators"
date: 2015-04-09 11:41:19 -0400
comments: true
categories: Generators Ruby
---

![Generators For Bunnies](http://i.imgur.com/O93mpLK.jpg =200x)

Happy Passover/Easter to all! In the spirit of the holidays, a short generators for bunnies blog post. (In other news, did everyone see the photos of [Obama with the Easter Bunny](http://www.bizpacreview.com/wp-content/uploads/2014/04/Obama-and-Easter-Bunny2.png)??)

Recently, I've realized just how often I `touch` a Ruby file and then start by defining methods within it. I'd love to save a little bit of time by building a Ruby file/methods generator – which will be a great exploration of generators and `ARGV`.

###`ARGV`

When you run a Ruby file on your computer, any arguments after `ruby` and the file name are stored in a variable called `ARGV`. For example, if I run `ruby file.rb what's up doc`, `ARGV` will equal `["what's", "up", "doc"]`. And `ARV[0]` will equal `"what's"`.

###Building the Program

The first challenge in my generator, which I'll call `bunny_methods.rb`, is to take the file name argument and `touch` the file as usual. There are a few different ways to do this: I'll go through the system, because I want more practice interacting with Terminal through my Ruby programs:
```ruby
file_name = ARGV[0]
system("touch #{file_name}")
```

So, `bunny_methods.rb` takes the first command-line argument – `ARGV[0]` – and creates a file with that name. But this file is still empty. Now I want to take the remaining command-line arguments and create empty methods inside of my new file.
```ruby
method_names = ARGV[1..-1]

open(file_name, 'w') do |f|
  method_names.each_with_index do |method, i|
    f << "\n\n" if i != 0
    f << "def #{method}\nend"
  end
end
```

If I run `ruby bunny_methods.rb new_file.rb method1 method2`, I get a new file `new_file.rb` with the contents:
```ruby
def method1
end

def method2
end
```

###Success!
![Bunnies!](http://media.giphy.com/media/1305AEqeMd0GNG/giphy.gif =300x)