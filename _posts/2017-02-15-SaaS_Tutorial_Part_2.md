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
