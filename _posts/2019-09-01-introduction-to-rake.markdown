---
layout: post
title:  "Introduction to Rake"
author: "Nathan Hennig"
date:   2019-05-23 15:30:00 -0700
tags:   ruby rails rake
---

Code version used while writing this article:
  * **Rails Version:**  5.2.2  
  * **Ruby Version:**   2.6.0-rc1

## What is Rake?

The word Rake is a portmanteau of Ruby and Make ([Make](https://en.wikipedia.org/wiki/Make_(software)) is one of the classic tools for building software).

*(The following text is copied from the [ruby/rake GitHub repository](https://github.com/ruby/rake))*


> Rake is a Make-like program implemented in Ruby. Tasks and dependencies are specified in standard Ruby syntax.
>
> Rake has the following features:
> 
> * Rakefiles (rake's version of Makefiles) are completely defined in standard Ruby syntax. No XML files to edit. No quirky > Makefile syntax to worry about (is that a tab or a space?)
> 
> * Users can specify tasks with prerequisites.
> 
> * Rake supports rule patterns to synthesize implicit tasks.
> 
> * Flexible FileLists that act like arrays but know about manipulating file names and paths.
> 
> * A library of prepackaged tasks to make building rakefiles easier. For example, tasks for building tarballs. (Formerly > tasks for building RDoc, Gems, and publishing to FTP were included in rake but they're now available in RDoc, RubyGems, > and rake-contrib respectively.)
> 
> * Supports parallel execution of tasks.

Any developer who uses Ruby on Rails will be familiar with at least a few Rails specific rake tasks. The ones I use the most frequently are `rake routes` and the `rake db` series. You can see all the rake tasks available to you using `rake --tasks`.

## Rake Hello World

Rake can be installed with `gem install rake`. If you're using Rails, it comes with Rake by default.

Rake tasks are kept in Rakefiles, so our first step is to create a `Rakefile` file. Once you have a Rakefile, go ahead and open it in your preferred text editor.


```ruby
task default: %w[hello_world]

task :hello_world do
  # ruby "hello.rb"
  puts "Hello World"
end
```

This Rakefile has two tasks. The first is `default` which has the `hello_world` task as a dependency. Using the `rake` command without any arguments will run the default task. Ruby files can be run using the `ruby "<path>"` method, but to save time I've just directly written `puts "Hello World"`.

## Conclusion

Rake is a great tool for any Ruby programmer. Whenever you find your self running a set of commands frequently, you should stop and ask yourself: "Could I write a Rake task for this?"


## Further Reading/References

* Ruby On Rails Guide - [Rake](https://guides.rubyonrails.org/v3.2/command_line.html#rake){:target="_blank"}
* GitHub - [ruby/rake](https://github.com/ruby/rake){:target="_blank"}
* Avdi Grimm's Rake Series - [Rake Basics](http://www.virtuouscode.com/2014/04/21/rake-part-1-basics/){:target="_blank"}
* Wikipedia - [Make](https://en.wikipedia.org/wiki/Make_(software)){:target="_blank"}
