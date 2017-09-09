---
layout:     post
title:      "SaaS Tutorial Part 2"
subtitle:   "Our first model"
date:       2017-02-16 12:00:00
author:     "Will Olson"
header-img: "img/post-bg-02.jpg"
---
## Intro
With this second post in the series, we are going to create the Accounts model, controller, views, routes, and spec tests. Throughout this series, I am going to be doing my best to follow the practices of Test Driven Development (TDD). This is a great way to keep your code reliable and if you are interested in learning more check out [this great tutorial series](https://everydayrails.com/2012/03/12/testing-series-intro.html).

Again, if you want to look at the source code for this post [look here!](https://github.com/frankolson/CoffeeTracking/tree/part_2)

---

## Outline
1. [Generating the Model](#section-1)
2. [Creating Our First Tests](#section-2)
3. [Implementing the Model](#section-3)
4. [Adding Specs, Controller, and Routes](#section-4)
5. [Finishing the Specs and Adding Our First Views](#section-5)

---

## The Good Stuff

### Generating the Model {#section-1}

First thing we are going to do is use the rails generators to create the Account model. With these next terminal commands we are going to generate the files necessary for the Account model, create and migrate our database, and finally start Guard to continuously keep on eye on our spec tests.

~~~
rails g model Account subdomain owner_id:integer
rails db:create db:migrate
bundle exec guard
~~~

### Creating Our First Tests {#section-2}

When we ran the rails generator to create our Account model, RSpec should have created an account factory as well as a model spec. Let's update that account factory. This will help us build testable Account ruby objects.

~~~ruby
# spec/factories/accounts.rb

FactoryGirl.define do
  factory :account do
    sequence(:subdomain) { |n| "subdomain#{n}"}
  end
end
~~~

In the following file, we use the FactoryGirl `sequence` command to make sure that we don't create any accounts with the same subdomain. We will test for this soon too. I should mention, in the first post of this series, I forgot to add the Shoulda-Matchers configuration to our rails helper. So right under `Capybara.app_host = 'http://example.com'` in the the `rails_helper.rb` file add the following.

~~~ruby
# spec/rails_helper.rb

Shoulda::Matchers.configure do |config|
  config.integrate do |with|
    with.test_framework :rspec
    with.library :rails
  end
end
~~~

Next let's create some tests to make sure we build out or model correctly. Here are some testable features we know about our Account model.

  1. All accounts have to have a unique lowercase based subdomain.
  2. All accounts need to eventually have an owner.
     - This gets tricky though because we don't have users yet, so we will just make a pending test for this one and come back to it.
  3. Subdomains should not have special characters.
  4. There should be restricted subdomains like `www`.

With these in mind we are going to create some tests that our model has to pass

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

### Implementing the Model {section-3}

If you have been keeping an eye on Guard you will notice that almost all of the tests fail. We are going to fix this right now. Let's jump into the previously generated Account model file and add in our validations.

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

Now all of our tests pass! Time to commit our changes.

~~~
git add .
git commit -m "create account model and associated tests"
~~~

### Adding Specs, Controller, and Routes {section-4}

Let's setup our controller specs now. For the accounts we are going to need specs for our `:new` and `:create` actions to start.

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

Looking at Guard it should be telling us that there is no `AccountsController` so lets create one ...

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

Sweet all of our tests are passing. Let's go ahead and commit our work again.

~~~
git add .
git commit -m "add initial account specs, controller, and routes"
~~~

### Finishing the Specs and Adding Our First Views {#section-5}

Now let's go complete those spec tests, and we are going to start with the `:new` action. _Side note: in the last post I forgot to add a gem to the development and test block of the Gemfile, so we're are going to fix that here as well._

~~~ruby
# Gemfile
group :development, :test do
  gem 'rails-controller-testing'
end
~~~

~~~
bundle install
~~~

~~~ruby
# spec/controllers/account_controller_spec.rb

it "assigns a new Account to @account" do
  get :new
  expect(assigns(:account)).to be_a_new(Account)
end
it "renders the :new template" do
  get :new
  expect(response).to render_template :new
end
~~~

Guard will tell us that that we don't have the right template for this action, so let's create one real quick, as well as fixing up the layout template and adding some CSS to work with bootstrap. We are also going to need to create a bootstrap flash messages helper to make our lives easier.

~~~sass
/* # app/assets/stylesheets/application.scss */

body {
  padding-top: 60px;
}
~~~

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

    <%= stylesheet_link_tag    'application'%>
    <%= javascript_include_tag 'application'%>
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

Guard is now telling us that we need to assign an new instance of Account in our controller action.

~~~ruby
# app/controllers/accounts_controller.rb

def new
  @account = Account.new
end
~~~

Now that those tests are passing lets move on to the create controller specs.

~~~ruby
# spec/controllers/account_controller_spec.rb

describe 'POST #create' do
  context "with valid attributes" do
    it "saves the new account in the database" do
      expect{
        post :create, params: { account: attributes_for(:account) }
      }.to change(Account,:count).by(1)
    end

    it "redirects to the root page" do
      post :create, params: { account: attributes_for(:account) }
      expect(response).to redirect_to(root_path)
    end
  end

  context "with invalid attributes" do
    it "does not save the new account" do
      expect{
        post :create, params: { account: { subdomain: '' } }
      }.to_not change(Account,:count)
    end

    it "re-renders the new method" do
      post :create, params: { account: { subdomain: '' } }
      expect(response).to render_template(:new)
    end
  end
end
~~~

We have to do a few things to fix these failing tests. First, add the create action functionality.

~~~ruby
# app/controllers/accounts_controller.rb

def create
  @account = Account.new(account_params)
  if @account.save
    redirect_to root_path, notice: "Your account was successfully created"
  else
    render :new
  end
end
~~~

Second, we have to create a welcome controller, its spec test, and the index view to act as a landing page for our app.

~~~ruby
# spec/controllers/welcome_controller_spec.rb

require 'rails_helper'

describe WelcomeController, type: :controller do
  describe 'GET #index' do
    it "renders the :index template" do
      get :index
      expect(response).to render_template(:index)
    end
  end
end
~~~

~~~ruby
# app/controllers/welcome_controller.rb

class WelcomeController < ApplicationController

  def index
  end

end
~~~

~~~erb
<!-- # app/views/welcome/index.html.erb -->

<div class="col-md-10 col-lg-6 offset-md-1 offset-lg-3 jumbotron">
  <div class="text-center">
    <h2>Welcome Coffee Tracker</h2>
    <%= link_to "Create an account", new_account_path, class: 'btn btn-lg btn-primary' %>
  </div>
</div>
~~~

Finally, we need to fix the routes by adding a root path.

~~~ruby
# config/routes.rb

root 'welcome#index'
~~~

Let's take a look at our new pages!

#### Landing page
![welcome index]({{ site.baseurl }}/img/saas_tutorial/welcome_index.png)

#### New Account Page
![account new]({{ site.baseurl }}/img/saas_tutorial/account_new.png)

Finally we shall finish this post by committing our work.

~~~
git add .
git commit -m "add accounts and welcome controller and associated specs/routes"
~~~
