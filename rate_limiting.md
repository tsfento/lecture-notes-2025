# Rate Limiting and Performance Metrics in Rails

## Introduction

- We are going to discuss rate limiting and how to implement it in a Rails application

## Live Coding

Rate limiting is a way to control the amount of incoming and outgoing traffic to or from a network. It is used to prevent abuse and ensure that the network runs smoothly.

What are examples of how users might abuse your application?

- Brute force attacks: Attempting to gain unauthorized access by trying numerous combinations of usernames and passwords.
- Scraping: Aggressively extracting data from your site, which can lead to bandwidth issues and degrade the experience for legitimate users.
- Denial of Service (DoS) attacks: Flooding your site with an overwhelming amount of traffic to make the service slow down or crash.
- Spamming: Posting or sending unsolicited messages or content in bulk.

This makes rate limiting an important part of your application.

Here are the pros of rate limiting

- Protects your application from abuse
- Ensures that your application runs smoothly
- Prevents your application from being overwhelmed

### Start

Create a new Rails api

```bash
rails new rate_limiting_rails_api --api
```

Change into the directory

```bash
cd rate_limiting_rails_api
```

Execute the following

```bash
rails g model User name:string email:string
```

```bash
rails db:migrate
```

Create a controller

```bash
rails g controller Users
```

Add the following to the `UsersController`

```ruby
class UsersController < ApplicationController
  def create
    user = User.new(user_params)
    if user.save
      render json: user, status: :created
    else
      render json: user.errors, status: :unprocessable_entity
    end
  end

  private

  def user_params
    params.permit(:name, :email)
  end
end
```

Add the following to the `config/routes.rb` file

```ruby
Rails.application.routes.draw do
  resources :users
end
```

### Adding Rate Limiting

To incorporate rate limiting into your application, you can use the `rack-attack` gem. This gem provides middleware to protect your application from abuse.

To use the `rack-attack` gem, add the following to your Gemfile:

```ruby
gem 'rack-attack'
```

Then run `bundle install` to install the gem.

In your `config/application.rb` file, add the following:

```ruby
config.middleware.use Rack::Attack
```

Create a file called `config/initializers/rack-attack.rb`. Here you can configure the rate limiting rules. Here is an example of how you can configure the rate limiting rules:

```ruby
class Rack::Attack
  Rack::Attack.cache.store = ActiveSupport::Cache::MemoryStore.new

  throttle('create_user/ip', limit: 1, period: 20.seconds) do |req|
    if req.path == '/users' && req.post?
      req.ip
    end
  end
end
```

Say that you want to limit the number of requests to the `/users` endpoint to 1 request every 20 seconds. You can use the `throttle` method to achieve this. Let's break this down:

```ruby
throttle
```

This is the method that you use to define the rate limiting rule. It takes in several arguments and a block to describe the rule.

```ruby
('create_user/ip', limit: 1, period: 20.seconds)
```

`create_user/ip` is the name of the rule. It is used to identify the rule in the logs. The `limit` and `period` are the rate limiting parameters. The `limit` is the maximum number of requests that can be made in the `period`. So in this example, you can only make 1 request every 20 seconds.

```ruby
do |req|
  if req.path == '/users' && req.post?
    req.ip
  end
end
```

This is the block that describes the rule. It takes in a `req` object, which is the request object. You can use this object to define the conditions for the rule. In this example, the rule is only applied to requests to the `/users` endpoint that are `POST` requests. The `req.ip` is used to identify the client making the request. This is used to track the number of requests made by the client.

We've also added a cache store to the `Rack::Attack`class. This is used to store the rate limiting data. In this example, we are using the `ActiveSupport::Cache::MemoryStore` to store the data in memory. You can use other cache stores like `Redis` or `Memcached` to store the data which is ideal for production environments.

Test this out by making a request to the `/users` endpoint. You should see that you can only make 1 request every 20 seconds. If you make more than 1 request, you should see a `429 Too Many Requests` response.

### Global Rate Limiting

You're probably wondering, am I supposed to do this to all my requests? The answer is no. You can apply a global rate limiter to all your requests.

Here is an example of how you can apply a global rate limiter to all your requests:

```ruby
class Rack::Attack
  Rack::Attack.cache.store = ActiveSupport::Cache::MemoryStore.new

  throttle('req/ip', limit: 5, period: 5.seconds) do |req|
    req.ip
  end

    throttle('create_user/ip', limit: 1, period: 20.seconds) do |req|
    if req.path == '/users' && req.post?
      req.ip
    end
  end
end
```

Here we are saying that we want to limit the number of requests to 5 requests every 5 seconds. This is a global rate limiter that applies to all requests. This is useful if you want to protect your application from abuse. Take the `limit` and `period` parameters into consideration. The `limit` is the maximum number of requests that can be made in the `period`.

When it comes to applying rate limiting to your application, you need to consider the following factors:

- Understand Usage Patterns
  - Before setting rate limits, analyze your application's traffic to understand normal usage patterns. Look for average request rates, peak periods, and the behavior of both typical and power users.
  - Account for Growth: Set limits that accommodate current usage patterns while also leaving room for growth. Anticipate increased activity as your user base grows or as your application's functionality expands.
- Differentiate Between User Types
  - Role-based Limiting: Consider implementing different rate limits for different types of users (e.g., unauthenticated users, regular users, premium users). This approach allows you to tailor the user experience according to the level of access or service each user tier is supposed to receive.
  - IP vs. User Account: Apply rate limits based on the user account for authenticated requests and fall back to IP-based limiting for unauthenticated requests. This can help differentiate between users sharing an IP address (e.g., users on the same network) and treat them fairly.
- Communicate with Users
  - API Documentation: Clearly document your rate limiting policies in your API documentation to set the right expectations for developers and users.
  - Feedback in Responses: Include information about rate limits in HTTP headers of your responses (e.g., X-RateLimit-Limit, X-RateLimit-Remaining, X-RateLimit-Reset), especially when a limit is approached or exceeded. This helps users understand their current rate limit status and adjust their usage accordingly.

### Conclusion

In conclusion, this is a basic lesson of rate limiting and it is an important part of your application. It helps to protect your application from abuse and ensures that your application runs smoothly. The `rack-attack` gem is a great way to incorporate rate limiting into your application. It provides middleware to protect your application from abuse. You can use it to define rate limiting rules and apply them to your application.

# Any questions?
