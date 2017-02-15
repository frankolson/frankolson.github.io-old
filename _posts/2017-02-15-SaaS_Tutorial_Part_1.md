---
layout:     post
title:      "SaaS Tutorial Part 1"
subtitle:   "Let's get started"
date:       2017-02-15 12:00:00
author:     "Frank Olson"
header-img: "img/post-bg-01.jpg"
---


~~~
rails new CoffeeTracking --database=postgresql --T
cd CoffeeTracking/
git init
git add . && git commit -m "init commit"
~~~

- remove comments from Gemfile
- remove turbolinks
- add testing suite gems

~~~ruby
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
~~~

- remove turbo links references

~~~javascript
/* # app/assets/javascripts/application.js */
/* get rid of this */
//= require turbolinks
~~~

change this:
~~~erb
<!-- # app/views/layouts/application.html.erb -->

<%= stylesheet_link_tag    'application', media: 'all', 'data-turbolinks-track': 'reload' %>
<%= javascript_include_tag 'application', 'data-turbolinks-track': 'reload' %>
~~~

to this:
~~~erb
<%= stylesheet_link_tag    'application' %>
<%= javascript_include_tag 'application' %>
~~~

- install gems

~~~
bundle install
rails g rspec:install
bundle exec guard init
~~~

- Edit the Rails helper to include Rspec, Capybara, DatabaseCleaner, and FactoryGirl. I usually use a `rails_helper.rb` file from an old project as a starter.

~~~ruby
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
~~~

- save work

~~~
git add .
git commit -m "setup testing suite"
~~~

- add gems for bootstrap

~~~ruby
# Gemfile
gem 'bootstrap', '~> 4.0.0.alpha6'
gem 'simple_form'
~~~

- install simple_form

~~~
rails g simple_form:install --bootstrap
~~~

- right now simpleform does not have its bootstrap generator up to date with Twitter Bootstrap 4. So, in order to take advantage of all the new changes in Bootstrap 4, replace the contents of `config/initializers/simple_form_bootstrap.rb` with the following

~~~ruby
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
~~~

- rename `app/assets/stylesheets/application.css` to `app/assets/stylesheets/application.scss`, delete all contents, and insert the following to include bootstrap assets.

~~~sass
/* # app/assets/stylesheets/application.scss */
// vendor assets
@import "bootstrap";

// project assets
~~~

- in `app/assets/javascripts/application.js` add `bootstrap-sprockets` to the list of requirements after `jquery_ujs`

~~~javascript
/* # app/assets/javascripts/application.js */

//= require jquery_ujs
//= require bootstrap-sprockets
~~~

- save work

~~~
git add .
git commit -m "install bootstrap and simpleform"
~~~
