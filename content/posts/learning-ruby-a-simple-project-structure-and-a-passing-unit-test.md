---
title: "Learning Ruby - a simple project structure and a passing unit test"
date: 2018-10-19T12:00:00+01:00
draft: false
---
Today, I started to learn Ruby. Learning a new language is simple — learn the syntax, constructs and over time pick up the idiomatic ways of doing things. Right? Not so much.

Yes, learning the language is important. My programming has been predominantly in statically typed languages, most notably Java. I have sufficient familiarity with the standard project structure for Java projects and the tooling to support dependency management and unit testing to be reasonably productive.

I decided that being able to get a simple “Hello, world!” unit test to pass would be a good approach to learn a very basic project structure and to start to learn the tools and the Ruby language. The purpose of this quick blog post is to serve as a reminder to me as I continue to learn the language.

The first step was to install Ruby. Rather than use [rbenv](https://github.com/rbenv/rbenv), I chose to use [asdf](https://github.com/asdf-vm/asdf) as my Ruby version manager. I use it for most of my other SDK’s, so I thought I’d try it out for Ruby too. Once Ruby was installed, I simply set a global version for use on my system — it’s just as simple to set a local version (per directory, aka. project).

With that out of the way, I created a new project directory:

```bash
mkdir -p ~/Projects/ruby/trial/{bin,lib,spec} && cd ~/Projects/ruby/trial
```

As can be seen from the above command, I’ve chosen to use [RSpec](http://rspec.info/) as my test library of choice. Before I can install RSpec, however, I need to configure a Gemfile:

```ruby
source "https://rubygems.org"

gem 'rspec', '~> 3.0'
```

I’ll use `bundler` to install RSpec as a dependency for the project, then initialise the project to use RSpec:

```bash
gem install bundler
bundle install --binstubs
bin/rspec --init
```

With the project structure set up, it’s time to get the trial project working. The first step, as always, is to write a failing test. This goes in `spec/trial_spec.rb`:

```ruby
require 'trial'

RSpec.describe Trial, '#hello' do
  context 'for a new project' do
    it 'greets the world' do
      trial = Trial.new
      expect(trial.hello).to eq 'Hello, world!'
    end
  end
end
```

Having saved the file, I can run the test, but this will fail due to lack of a Trial class and hello method. I’ll create a quick stub in `lib/trial.rb` so that the test will compile and run, but give me a failing result (red):

```ruby
class Trial
  def hello
  end
end
```

I can now run the test and see it fail:

```bash
bin/rspec --format doc
```

The next step is to make the test pass (green):

```ruby
class Trial
  def hello
    "Hello, world!"
  end
end
```

Running the test again, it now passes. Excellent, there’s no refactoring to do (arguably, I could `require 'rspec'` in `trial_spec.rb` and change `RSpec.describe...` to `describe...` but I don’t think it’s worth it in this case). I now have a working unit test and a basic project structure:

```bash
**.
**├── Gemfile
├── Gemfile.lock
├── **bin
**│   ├── **bundle
**│   ├── **htmldiff
**│   ├── **ldiff
**│   └── **rspec
**├── **lib
**│   └── trial.rb
└── **spec
    **├── spec_helper.rb
    └── trial_spec.rb
```

I think that’s a good outcome and provides me with a solid starting point to really start getting to grips with the Ruby language.
