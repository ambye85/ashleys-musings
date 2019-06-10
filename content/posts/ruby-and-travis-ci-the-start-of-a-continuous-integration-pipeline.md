---
title: "Ruby and Travis CI - the start of a continuous integration pipeline"
date: 2018-10-24T12:00:00+01:00
draft: false
---
I’m about to embark on writing the venerable game of TicTacToe in Ruby. Given that I want to develop in a way that enables each of the 3 tenets of DevOps to be achieved, I need to configure the project for Continuous Integration (CI).

CI is important in DevOps because it helps to optimise development for speed, which increases flow. By optimising for speed, quality should be guaranteed because poor quality code will break and decrease speed. CI also aids in fast feedback when combined with Test Driven Development (TDD) — in fact, TDD is a prerequisite for achieving fast flow too. If a test fails, the build fails and I will be alerted to this. Once these two tenets (fast flow and fast feedback) have been achieved, the stage is set for being able to undertake the third: continual experimentation and learning.

To use CI, all project source files (code and any infrastructure that I need) need to be kept in version control. The rest of this article will explain how to set up this initial repository and show the build and code coverage status of a simple unit test.

First, set up a basic Ruby project. I wrote about this in a [previous blog post](https://medium.com/@ambye/learning-ruby-a-simple-project-structure-and-a-passing-unit-test-ca4cb6af00ce). I want to display a test coverage status at the top of the project’s README. I am going to use [SimpleCov](https://github.com/colszowka/simplecov) and [Codecov](https://codecov.io/) to do this, and it requires the use of two additional Ruby Gems. Edit the Gemfile:

```ruby
source 'https://rubygems.org'

gem 'rspec', '~> 3.8.0'
gem 'simplecov', require: false, group: :test
gem 'codecov', require: false, group: :test
```

Install the Gems with bundle install. Now edit `./spec/spec_helper.rb` and add the following lines to the very top of the file:

```ruby
require 'simplecov'
SimpleCov.start

require 'codecov'
SimpleCov.formatter = SimpleCov::Formatter::Codecov
```

These ensure that code coverage is required before any other files are required. The results are also formatted in a manner compatible with Codecov.

Next, create a simple test that we will watch fail and then make pass — the purpose of this test is to make sure everything is configured correctly. Create and edit `./spec/setup_spec.rb`:

```ruby
require 'rspec'
require 'setup'

describe Setup do
  it "passes" do
    expect(Setup.new().check()).to eq true
  end
end
```

Before running this test, also create the `Setup` class so that we avoid compiler errors. Edit `./lib/setup.rb`:

```ruby
class Setup
  def check()
    false
  end
end
```

Run the test with `bundle exec rspec --format doc` and note that it fails. Now amend the test to make it pass:

```ruby
class Setup
  def check()
    true
  end
end
```

Issuing the command `open ./coverage/index.html` should open a web browser with the coverage results for the above test. Close that browser window.

The final step needed to make the CI work is to create a configuration file for [Travis CI](https://travis-ci.org/) (there are other CI services that could be used). In the root of the project, create and edit `.travis.yml`:

```ruby
language: ruby
rvm:
  - 2.5.2
install: bundle install
script:
  - bundle exec rspec --format doc
  ```

The configuration file specified the language name and tells RVM which Ruby version to use (I’m not using RVM so this needs to be set here; if my project had a `.ruby-version` file, Travis is capable of determining the correct Ruby version from this file). Next it modifies the default install command to install our required Gems. Lastly, it executes the RSpec tests. By using GitHub, Travis CI and Codecov, the coverage results are automatically uploaded for our project.

Since my basic Ruby project structure doesn’t contain a `README.md` file, add that now. Create a new repository in GitHub, add the project files to the commit and push an initial commit. If everything is working correctly, Travis should display a passing build state for the repository and Codecov should show the test coverage status.

To display these states in the README we need to ass badges from Travis and Codecov. The required markdown for each of these can be obtained from the relevant project page for each of the two tools. Add these to the top of README.md — here’s mine for the TicTacToe game:

    [![Build Status](https://travis-ci.org/ambye85/tictactoe-ruby.svg?branch=master)](https://travis-ci.org/ambye85/tictactoe-ruby)
    [![codecov](https://codecov.io/gh/ambye85/tictactoe-ruby/branch/master/graph/badge.svg)](https://codecov.io/gh/ambye85/tictactoe-ruby)

    # TicTacToe (Ruby Edtion)

    The venerable game of TicTacToe, written in Ruby.

Commit the updated file and push to version control. A second CI build will run and once complete the status will be visible in the README:

![](https://cdn-images-1.medium.com/max/2000/1*_uEYCiQyPk6jVcVu7hm2Bw.png)

Check out the relevant commit in my [tictactoe-ruby repository](https://github.com/ambye85/tictactoe-ruby/tree/bff44843bfb85a2967bcae413cfdee21eb7a1ed7) to view how this configuration looks for me.
