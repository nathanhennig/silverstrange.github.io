---
layout: post
title:  "My First Ruby Gem"
author: "Nathan Hennig"
date:   2019-04-17 22:33:00 -0700
tags:   ruby gem
---
 <a href="https://github.com/SilverStrange/hello_world/tree/ruby/gem" class="btn btn-github"><span class="icon"></span>View Ruby Gem on GitHub</a>

Code version used while writing this article:
  * **Ruby Version:**   2.6.1

A Ruby gem is prepackaged code that can easily installed and used. Java's jars and Python's packages can be considered equivalents of Ruby's gems. RubyGems.org is the community's gem hosting service. Most Ruby gems are hosted here and any gems that are can be installed automatically by Ruby with a simple `gem install gemname` (at least from Ruby 1.9.2 and onwards).

I've used many gems over the course of my Ruby on Rails work, but I'd never gotten around to writing my own gem before.

Today I will run through the RubyGems guide: [Make Your Own Gem](https://guides.rubygems.org/make-your-own-gem/) and make a simple 'Hello World' gem.

***
First you need a name for your gem. If you want to publish it on RubyGems.org, make sure the gem name is unique. Since I don't want to publish my gem, I'll just call it hola like the guide does.

The basic structure for a gem is:
```
% tree
.
|── hola.gemspec
└── lib
    └── hola.rb
```
The gemspec provides information about the gem, which will be used for the gem's RubyGems homepage (if published). A wide variety of information can included here, but the most important information is probably the version number and any runtime dependencies your gem has.

My gemspec:
```ruby
Gem::Specification.new do |s|
  s.name        = 'hola'
  s.version     = '0.0.0'
  s.date        = '2019-04-16'
  s.summary     = "Hola!"
  s.description = "A simple hello world gem"
  s.authors     = ["Nathan Hennig"]
  s.email       = 'nathan.hennig@gmail.com'
  s.files       = ["lib/hola.rb"]
  s.homepage    =
    'https://github.com/SilverStrange/hello_world/tree/ruby/gem'
  s.license       = 'MIT'
end
```

For the actual hello world portion all you need is a class with a class method in `lib/hola.rb`.
```ruby
class Hola
  def self.hi
    puts "Hello world!"
  end
end
```

Now the gem is ready to be built and installed.
```
gem build hola.gemspec
gem install ./hola-0.0.0.gem
```

Then we can load up irb and verify that it works.
```ruby
irb
require 'hola'
Hola.hi
"Hello World!"
 => nil
```

## Closing Thoughts
I was expecting this to be very simple, but I was impressed by how little was required to create a basic 'Hello World' gem. There is plenty of room for complexity when including more code and various gems to help with development such as [Rspec](http://rspec.info/) and [RuboCop](https://docs.rubocop.org/en/latest/), but the most basic gem only needs two files.

**Further Reading:**
  * [Developing a RubyGem Using Bundler](https://bundler.io/v1.13/guides/creating_gem)
  * [RubyGems Guides](https://guides.rubygems.org/)
