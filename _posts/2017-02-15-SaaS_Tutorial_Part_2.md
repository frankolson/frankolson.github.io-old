---
layout:     post
title:      "SaaS Tutorial Part 2"
subtitle:   "Our first model"
date:       2017-02-16 12:00:00
author:     "Frank Olson"
header-img: "img/post-bg-02.jpg"
---
## Intro

---

## Outline

---

## The Good Stuff

- create model with string attribute `subdomain`
- create your data base if you have not already, then migrate said database.
- start guard, we about to do some testing!

~~~
rails g model Account subdomain owner_id:integer
rails db:create db:migrate
bundle exec guard
~~~

- when we ran the rails generator to create our Account model, rspec should have created an account factory as well as a model spec.
- Let's start with the Factory. This will help us build testable Account ruby objects.

~~~ruby
# spec/factories/accounts.rb

FactoryGirl.define do
  factory :account do
    sequence(:subdomain) { |n| "subdomain#{n}"}
  end
end
~~~

- In the following file, we use the FactoryGirl `sequence` command to make sure that we don't create and accounts with the same subdomain. We will test for this soon too.

- Next let's create some tests to make sure we build out or model correctly.
- Here are some testable features we know about our Account model.
  1. all accounts have to have a unique lowercase based subdomain
  2. all accounts need to eventually have an owner
    - this gets tricky though because we don't have users yet, so we will just make a pending test for this one and come back to it.
  3. subdomains should not have special characters
  4. there should be restricted subdomains like `www`
- with these in mind we are going to create some tests that our model has to pass

~~~ruby
# spec/models/account_spec.rb

require 'rails_helper'

RSpec.describe Account, type: :model do
  describe 'validations' do
    it { should validate_presence_of(:subdomain) }
    it { should validate_uniqueness_of(:subdomain).case_insensitive }

    it { should allow_value('franko').for(:subdomain) }
    it { should allow_value('test').for(:subdomain) }

    it { should_not allow_value('www').for(:subdomain) }
    it { should_not allow_value('WWW').for(:subdomain) }
    it { should_not allow_value('.test').for(:subdomain) }
    it { should_not allow_value('test/').for(:subdomain) }

    it 'should validate case insensitive uniqueness' do
      create(:account, subdomain: 'Test')
      expect(build(:account, subdomain: 'test')).to_not be_valid
    end
  end

  describe 'associations' do
    it "belongs to an owner"
  end
end
~~~

- If you have been keeping an eye on Guard you will notice that almost all of the tests fail.
- We are going to fix this right now. Let's jump into the previously generated Account mdel file and add in our validations.

~~~ruby
class Account < ApplicationRecord
  RESTRICTED_SUBDOMAINS = %w(www)

  # Validations
  validates :subdomain, presence: true,
                        uniqueness: { case_sensitive: false },
                        format: { with: /\A[\w\-]+\Z/i, message: 'contains invalid characters' },
                        exclusion: { in: RESTRICTED_SUBDOMAINS,  message: 'restricted' }
  before_validation :downcase_subdomain

  private

  def downcase_subdomain
    self.subdomain = subdomain.try(:downcase)
  end
end
~~~

- Now all of our tests pass!
- Time for a save point.

~~~
git add .
git commit -m "create account model and associated tests"
~~~

- let's setup our controller specs now.
- for the accounts we are going to need specs for our new and create actions to start
- lets create the controllers spec file

~~~ruby
# spec/controllers/account_controller_spec.rb

require 'rails_helper'

describe AccountsController, type: :controller do
  describe 'GET #new' do
    it "assigns a new Account to @account"
    it "renders the :new template"
  end

  describe 'POST #create' do
    context "with valid attributes" do
      it "saves the new contact in the database"
      it "redirects to the home page"
    end

    context "with invalid attributes" do
      it "does not save the new contact in the database"
      it "re-renders the :new template"
    end
  end
end
~~~

- looking at guard it should be telling us that there is no AccountsController so lets create one ...

~~~ruby
# app/controllers/accounts_controller.rb

class AccountsController < ApplicationController

  def new
  end

  def create
  end

  private

  def account_params
    params.require(:account).permit(:subdomain)
  end
end
~~~

... and add the routes to the routes file.

~~~ruby
# config/routes.rb

Rails.application.routes.draw do
  resources :accounts, only: [:new, :create]
end
~~~

- Sweet all of our tests are passing. Let's go ahead and commit our work again.

~~~
git add .
git commit -m "add initial account specs, controller, and routes"
~~~

- Now let's go complete those spec tests
- And we are going to start with the new action.

~~~ruby
# spec/controllers/account_controller_spec.rb

it "assigns a new Account to @account" do
  get :new
  expect(assigns(:account)).to be_a_new(Account)
end
~~~

- Guard will tell us that that we don't have the right template for this action, so let's create one real quick as well as fixing up the layout template to work with bootstrap.
- We are also going to need to create a bootstrap flash messages helper to make our lives easier.

~~~ruby
# app/helpers/application_helper.rb

module ApplicationHelper
  def bootstrap_class_for flash_type
    { success: "alert-success", error: "alert-danger", alert: "alert-warning", notice: "alert-info" }[flash_type.to_sym] || flash_type.to_s
  end

  def flash_messages(opts = {})
    flash.each do |msg_type, message|
      concat(content_tag(:div, message, class: "alert #{bootstrap_class_for(msg_type)} alert-dismissible fade show") do
              concat content_tag(:button, 'x', class: "close", data: { dismiss: 'alert' })
              concat message
            end)
    end
    nil
  end
end
~~~

~~~erb
<!-- # app/views/layouts/application.html.erb -->

<!DOCTYPE html>
<html>
  <head>
    <title>CoffeeTracking</title>
    <%= csrf_meta_tags %>

    <%= stylesheet_link_tag    'application', media: 'all', 'data-turbolinks-track': 'reload' %>
    <%= javascript_include_tag 'application', 'data-turbolinks-track': 'reload' %>
  </head>

  <body>

    <div class="container">
      <%= flash_messages %>

      <div class="row">
        <%= yield %>
      </div>
    </div>

  </body>
</html>
~~~

~~~erb
<!-- # app/views/accounts/new.html.erb -->

<div class="col-md-10 col-lg-6 offset-md-1 offset-lg-3 card card-block">
  <h2>Sign up for Coffee Tracker</h2>
  <%= simple_form_for @account do |f| %>
      <%= f.input :subdomain do %>
        <div class="input-group">
          <%= f.input_field :subdomain, placeholder: 'dublin', class: "form-control" %>
          <span class="input-group-addon">.coffeetracker.io</span>
        </div>
      <% end %>
    <%= f.button :submit, class:"btn-primary" %>
  <% end %>
</div>
~~~