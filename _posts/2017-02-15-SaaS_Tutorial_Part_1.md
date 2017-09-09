---
layout:     post
title:      "SaaS Tutorial Part 1"
subtitle:   "Let's get started"
date:       2017-02-15 22:14:00
author:     "Will Olson"
header-img: "img/post-bg-01.jpg"
---

## Intro

This is the first article of my series on how to build a Software as a Service (SaaS) web app using Ruby on Rails. In this series we are going to be setting up a multi-tenant web app to track employee satisfaction at individual stores in a large coffee franchise. Employees will be able to sign in and give reviews and store managers will be able to track the anonymized satisfaction of their employees.

Their are a few assumptions I am going to make about you, the reader / up-and-coming developer, and your setup.

1. You are currently using:
  - Rails version 5+
  - Ruby version 2.3.3+
  - A shell based terminal (I am using Linux Bash)
2. You are familiar with using RSpec but not necessarily and expert.
3. You are familiar with Omniauthable Devise but not necessarily an expert.

In this first post we are going to focus on setting up our rails app with a great rspec testing suite as well as Twitter bootstrap styling. If you want to look at the source code for this post [look here!](https://github.com/frankolson/CoffeeTracking/tree/part_1)

---

## Outline

1. [Building the App](#section-1)
2. [Adding the First Gems](#section-2)
3. [Configuring the RSpec Suite](#section-3)
4. [Add the Style](#section-4)

---

## The Good Stuff

### Building the App {#section-1}

To start we are going to build our app using PostgreSQL as our database using `--database=postgresql`. This is really important for two reasons. First, we are going to host our site on Heroku for easy deployment which requires that we use PostgreSQL. Second, and most important, we are going to utilize schemas to implement multi-tenancy which is only found in PostgreSQL.

Also, we are going to use the `-T` option to have Rails ignore the default unit test generator. For this app we will be using RSpec to handle the unit (or spec) tests.
```
rails new CoffeeTracking --database=postgresql -T
cd CoffeeTracking/
```

Also, because we're awesome developers, we are going to Git to track our development with version control.
```
git init
git add . && git commit -m "init commit"
```

### Adding the First Gems {#section-2}

Next, we are going to add all the gems for our testing suite. RSpec for unit tests, Guard for continuous testing, FactoryGirl for model factories, and should-matchers to make our lives easier. Also, if you notice I have removed all the unnecessary comments as well as the Turbolinks gem. We won't be needing it for this tutorial.

```ruby
# Gemfile
source 'https://rubygems.org'

git_source(:github) do |repo_name|
  repo_name = "#{repo_name}/#{repo_name}" unless repo_name.include?("/")
  "https://github.com/#{repo_name}.git"
end

gem 'rails', '~> 5.0.1'
gem 'pg', '~> 0.18'
gem 'puma', '~> 3.0'
gem 'sass-rails', '~> 5.0'
gem 'uglifier', '>= 1.3.0'
gem 'coffee-rails', '~> 4.2'

gem 'jquery-rails'
gem 'jbuilder', '~> 2.5'


group :development, :test do
  gem 'byebug', platform: :mri

  # Below in this block are the gems for the testing suite
  gem 'guard'
  gem 'guard-livereload'
  gem 'guard-rspec'
  gem 'rspec-rails'
  gem 'capybara'
  gem 'factory_girl_rails'
  gem 'database_cleaner'
  gem 'shoulda-matchers'
end

group :development do
  gem 'web-console', '>= 3.3.0'
  gem 'listen', '~> 3.0.5'
  gem 'spring'
  gem 'spring-watcher-listen', '~> 2.0.0'
end

gem 'tzinfo-data', platforms: [:mingw, :mswin, :x64_mingw, :jruby]
```

Because we removed the Turbolinks gem, we also need to remove references to it in the asset pipeline ...

```javascript
/* # app/assets/javascripts/application.js */
/* get rid of this */
//= require turbolinks
```

... as well as the layout view. Change this:

```erb
<!-- # app/views/layouts/application.html.erb -->

<%= stylesheet_link_tag    'application', media: 'all', 'data-turbolinks-track': 'reload' %>
<%= javascript_include_tag 'application', 'data-turbolinks-track': 'reload' %>
```

to this:
```erb
<%= stylesheet_link_tag    'application' %>
<%= javascript_include_tag 'application' %>
```

### Configuring the RSpec Suite {#section-3}

Now, that the gems have been added to the Gemfile, the need to be installed and configured. Here we run bundle install then generate the default configuration files for both RSpec and Guard.

```
bundle install
rails g rspec:install
bundle exec guard init
```

Once those files have been generated we now need to edit the Rails helper to include Rspec, Capybara, DatabaseCleaner, and FactoryGirl. I usually use a `rails_helper.rb` file from an old project as a starter.

```ruby
# spec/rails_helper.rb
ENV['RAILS_ENV'] ||= 'test'

require File.expand_path('../../config/environment', __FILE__)
abort("The Rails environment is running in production mode!") if Rails.env.production?
require 'spec_helper'
require 'rspec/rails'
require 'database_cleaner'
require 'capybara/rspec'

Dir[Rails.root.join("spec/support/**/*.rb")].each { |f| require f }

ActiveRecord::Migration.maintain_test_schema!

Capybara.app_host = 'http://lvh.me:3000'

RSpec.configure do |config|
  config.include FactoryGirl::Syntax::Methods
  config.include Rails.application.routes.url_helpers
  config.include Capybara::DSL

  config.before(:suite) do
    DatabaseCleaner.strategy = :transaction
    DatabaseCleaner.clean_with(:truncation)
  end
  config.before(:each) do
    DatabaseCleaner.start
  end
  config.after(:each) do
    DatabaseCleaner.clean
  end
end
```

Time to create a save point with Git!

```
git add .
git commit -m "setup testing suite"
```

### Add the Style {#section-4}

Its time to add some style, specifically with Twitter Bootstrap and SimpleForm. Go ahead and add then gems to the Gemfile ...

```ruby
# Gemfile
gem 'bootstrap', '~> 4.0.0.alpha6'
gem 'simple_form'
```

... and then generate the initializer files needed for SimpleForm.

```
rails g simple_form:install --bootstrap
```

Right now SimpleForm does not have its Bootstrap generator up to date with Twitter Bootstrap 4. So, in order to take advantage of all the new changes in Bootstrap 4, replace the contents of `config/initializers/simple_form_bootstrap.rb` with the following:

```ruby
# config/initializers/simple_form_bootstrap.rb
# Use this setup block to configure all options available in SimpleForm.
SimpleForm.setup do |config|
  config.error_notification_class = 'alert alert-danger'
  config.button_class = 'btn btn-default'
  config.boolean_label_class = nil
  config.wrappers :vertical_form, tag: 'div', class: 'form-group', error_class: 'has-danger' do |b|
    b.use :html5
    b.use :placeholder
    b.optional :maxlength
    b.optional :pattern
    b.optional :min_max
    b.optional :readonly
    b.use :label, class: 'form-control-label'
    b.use :input, class: 'form-control'
    b.use :error, wrap_with: { tag: 'span', class: 'form-control-feedback' }
    b.use :hint,  wrap_with: { tag: 'p', class: 'form-control-feedback' }
  end
  config.wrappers :vertical_file_input, tag: 'div', class: 'form-group', error_class: 'has-danger' do |b|
    b.use :html5
    b.use :placeholder
    b.optional :maxlength
    b.optional :readonly
    b.use :label, class: 'form-control-label'
    b.use :input
    b.use :error, wrap_with: { tag: 'span', class: 'form-control-feedback' }
    b.use :hint,  wrap_with: { tag: 'p', class: 'form-control-feedback' }
  end
  config.wrappers :vertical_boolean, tag: 'div', class: 'form-group', error_class: 'has-danger' do |b|
    b.use :html5
    b.optional :readonly
    b.wrapper tag: 'div', class: 'checkbox' do |ba|
      ba.use :label_input
    end
    b.use :error, wrap_with: { tag: 'span', class: 'form-control-feedback' }
    b.use :hint,  wrap_with: { tag: 'p', class: 'form-control-feedback' }
  end
  config.wrappers :vertical_radio_and_checkboxes, tag: 'div', class: 'form-group', error_class: 'has-danger' do |b|
    b.use :html5
    b.optional :readonly
    b.use :label, class: 'form-control-label'
    b.use :input
    b.use :error, wrap_with: { tag: 'span', class: 'form-control-feedback' }
    b.use :hint,  wrap_with: { tag: 'p', class: 'form-control-feedback' }
  end
  config.wrappers :horizontal_form, tag: 'div', class: 'form-group', error_class: 'has-danger' do |b|
    b.use :html5
    b.use :placeholder
    b.optional :maxlength
    b.optional :pattern
    b.optional :min_max
    b.optional :readonly
    b.use :label, class: 'col-sm-3 form-control-label'
    b.wrapper tag: 'div', class: 'col-sm-9' do |ba|
      ba.use :input, class: 'form-control'
      ba.use :error, wrap_with: { tag: 'span', class: 'form-control-feedback' }
      ba.use :hint,  wrap_with: { tag: 'p', class: 'form-control-feedback' }
    end
  end
  config.wrappers :horizontal_file_input, tag: 'div', class: 'form-group', error_class: 'has-danger' do |b|
    b.use :html5
    b.use :placeholder
    b.optional :maxlength
    b.optional :readonly
    b.use :label, class: 'col-sm-3 form-control-label'
    b.wrapper tag: 'div', class: 'col-sm-9' do |ba|
      ba.use :input
      ba.use :error, wrap_with: { tag: 'span', class: 'form-control-feedback' }
      ba.use :hint,  wrap_with: { tag: 'p', class: 'form-control-feedback' }
    end
  end
  config.wrappers :horizontal_boolean, tag: 'div', class: 'form-group', error_class: 'has-danger' do |b|
    b.use :html5
    b.optional :readonly
    b.wrapper tag: 'div', class: 'col-sm-offset-3 col-sm-9' do |wr|
      wr.wrapper tag: 'div', class: 'checkbox' do |ba|
        ba.use :label_input
      end
      wr.use :error, wrap_with: { tag: 'span', class: 'form-control-feedback' }
      wr.use :hint,  wrap_with: { tag: 'p', class: 'form-control-feedback' }
    end
  end
  config.wrappers :horizontal_radio_and_checkboxes, tag: 'div', class: 'form-group', error_class: 'has-danger' do |b|
    b.use :html5
    b.optional :readonly
    b.use :label, class: 'col-sm-3 form-control-label'
    b.wrapper tag: 'div', class: 'col-sm-9' do |ba|
      ba.use :input
      ba.use :error, wrap_with: { tag: 'span', class: 'form-control-feedback' }
      ba.use :hint,  wrap_with: { tag: 'p', class: 'form-control-feedback' }
    end
  end
  config.wrappers :inline_form, tag: 'div', class: 'form-group', error_class: 'has-danger' do |b|
    b.use :html5
    b.use :placeholder
    b.optional :maxlength
    b.optional :pattern
    b.optional :min_max
    b.optional :readonly
    b.use :label, class: 'sr-only'
    b.use :input, class: 'form-control'
    b.use :error, wrap_with: { tag: 'span', class: 'form-control-feedback' }
    b.use :hint,  wrap_with: { tag: 'p', class: 'form-control-feedback' }
  end
  config.wrappers :multi_select, tag: 'div', class: 'form-group', error_class: 'has-danger' do |b|
    b.use :html5
    b.optional :readonly
    b.use :label, class: 'form-control-label'
    b.wrapper tag: 'div', class: 'form-inline' do |ba|
      ba.use :input, class: 'form-control'
      ba.use :error, wrap_with: { tag: 'span', class: 'form-control-feedback' }
      ba.use :hint,  wrap_with: { tag: 'p', class: 'form-control-feedback' }
    end
  end
  # Wrappers for forms and inputs using the Bootstrap toolkit.
  # Check the Bootstrap docs (http://getbootstrap.com)
  # to learn about the different styles for forms and inputs,
  # buttons and other elements.
  config.default_wrapper = :vertical_form
  config.wrapper_mappings = {
    check_boxes: :vertical_radio_and_checkboxes,
    radio_buttons: :vertical_radio_and_checkboxes,
    file: :vertical_file_input,
    boolean: :vertical_boolean,
    datetime: :multi_select,
    date: :multi_select,
    time: :multi_select
  }
end
```

Then rename `app/assets/stylesheets/application.css` to `app/assets/stylesheets/application.scss` (focus on the file extension), delete all its content, and insert the following to include bootstrap assets.

```sass
/* # app/assets/stylesheets/application.scss */
// vendor assets
@import "bootstrap";

// project assets
```

In `app/assets/javascripts/application.js` add `bootstrap-sprockets` to the list of requirements after `jquery_ujs`.

```javascript
/* # app/assets/javascripts/application.js */

//= require jquery_ujs
//= require bootstrap-sprockets
```

And finally, create another save point.

```
git add .
git commit -m "install bootstrap and simpleform"
```

---

Next time we will be setting up the accounts model for our multi-tenant system.
