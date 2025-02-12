# The Ruby Web Project Setup List

Use this list to set up a project with:

- Sinatra and Rack for handling requests and responses
- Capybara for feature testing
- RSpec for unit testing

The result of following this guide is available for cloning from [this commit](https://github.com/sjmog/bookmark_manager/commit/a63f3aaabe43be3a55c1b4dd5e7b3cde4673d60f).

### 1. Ready a project for Sinatra, RSpec, and Capybara

Any Ruby project starts by creating a Gemfile, which lists all libraries (dependent programs, or 'gems') of this project. [Bundler](http://bundler.io/), which is a dependency management program for Ruby, can do this for us. Use Bundler's `init` command:

```sh
$> bundle init
```

List the gems in the Gemfile:

```ruby
# in Gemfile
source 'https://rubygems.org'

gem 'sinatra'
gem 'sinatra-contrib'
gem 'rspec'
gem 'capybara'
gem 'webrick'
```

We've listed dependencies as strings, but the actual programs aren't installed to the project yet. Bundler can look through a Gemfile, and install each gem listed there by executing the `gem` function with the provided string as an argument. Use Bundler's `install` command:

```sh
$> bundle install
```

### 2. Set up RSpec

```
.
├── spec
│   └── spec_helper.rb
└── .rspec
```

For `rspec` to work, we need two things:

-  a `spec` directory containing a `spec_helper.rb` file, which configures the test suite. The `spec_helper.rb` is automatically run every time you run `rspec`.
- a `.rspec` file, which configures the output of the `rspec` command itself. [`rspec` has modifier flags](https://relishapp.com/rspec/rspec-core/docs/command-line), like `--warnings`, which change the output of running `rspec` (but don't change the tests themselves).

Running RSpec with the `--init` flag will create these things for us:

```sh
$> rspec --init
```

### 3. Set up Sinatra and Rack

In the root directory, set up a basic Sinatra application in file called `app.rb`:

```ruby
# in app.rb

require 'sinatra/base'
require 'sinatra/reloader'

class BookmarkManager < Sinatra::Base
  configure :development do
    register Sinatra::Reloader
  end

  get '/' do
    'Hello World'
  end

  run! if app_file == $0
end
```

And configure the `rackup` command to run the application in `app.rb`, via a file called `config.ru`:

```ruby
# in config.ru

require_relative "./app"

run RPS
```

Your directory should now look like this:

```
.
├── Gemfile
├── Gemfile.lock
├── app.rb
├── config.ru
└── spec
    ├── features
    └── spec_helper.rb
```

### 4. Make Capybara talk to Sinatra

To configure Capybara, we need to adjust the `spec_helper.rb`, which configures our tests. Specifically, when we run `rspec` we need it to be configured in the following way:

- Set the environment to "test".
- Bring in the contents of the `app.rb` file.
- Require all the testing gems (RSpec, Capybara, and the Capybara/RSpec package that lets them talk to each other).
- Tell Capybara that any instructions like `visit('/')` should be directed at the application called 'BookmarkManager'.

Add the following to `spec/spec_helper.rb`:

```ruby
# at the top of spec/spec_helper.rb

# Set the environment to "test"
ENV['RACK_ENV'] = 'test'

# Bring in the contents of the `app.rb` file. The below is equivalent to: require_relative '../app.rb'
require File.join(File.dirname(__FILE__), '..', 'app.rb')

# Require all the testing gems
require 'capybara'
require 'capybara/rspec'
require 'rspec'

# Tell Capybara to talk to RPS
Capybara.app = RPS

### the rest of the file ###
```

> Obviously, you don't need to add the comments. But you should understand what each line does, so you can vary them later with more advanced packages and setup scenarios.

### 5. Profit!

You can now add feature tests to the `spec/features` directory, and run feature tests with `rspec spec/features`.

> Running `rspec` will run both features and regular unit specs.
