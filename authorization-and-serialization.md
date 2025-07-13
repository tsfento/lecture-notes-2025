# Authorization and Serialization

## Introduction

-   We are going to discuss authentication
-   We are going to discuss serialization
-   We are going to be adding both to our api

> [Codelabs Learning Assistant Demo](https://chatgpt.com/g/g-68484cbcb348819181c3f4137b0b7c49-codelabs-learning-assistant)
>
> - Ask the Codelabs Learning Assistant: “Can you give me an analogy that explains authentication vs. authorization?”

## Key Concepts
  - Authentication is verifying a user's identity.
  - Authorization determines what an authenticated user is allowed to do.
  - Serialization in Rails involves converting data into a format suitable for storage or transmission.

### Authentication vs. Authorization:
  - Authentication: Verifies who the user is (e.g., using username and password).
  - Authorization: Determines access rights after a user is authenticated (e.g., role-based access).

### Methods of Authentication:
  - Username and Password.
  - Two-Factor Authentication (2FA).
  - OAuth for logging in using third-party services.
  - Token-Based Authentication.
  - Single Sign-On (SSO).
  - Passwordless Authentication.

### Best Practices for Authentication:
  - Use strong, hashed passwords (e.g., BCrypt).
  - Implement 2FA.
  - Lock accounts after multiple failed login attempts.
  - Use HTTPS for secure data transmission.
  - Store sensitive data securely.
  - Keep software and libraries updated.
  - Enforce password complexity and length.
  - Secure password recovery processes.

### Best Practices for Authorization:
  - Grant minimal necessary permissions.
  - Use a role-based system for easier permission management.
  - Regularly audit user roles and permissions.
  - Consider context and time-based restrictions.

### Authentication in Rails:
  - Rails uses has_secure_password for a simple authentication system.
  - Use BCrypt for password hashing.
  - Creating a JWT (JSON Web Token):
  - Used for secure data transmission and authentication.
  - Consists of Header, Payload, and Signature.

### Authenticating and Verifying JWT:
  - Use JWTs for stateless authentication.
  - Verify JWTs to ensure data integrity and authenticity.

### Serialization with Blueprinter:
  - Blueprinter is used for efficient data serialization.
  - Control what data is sent and how it's formatted.
  - Create different views for different data representations.
  - Serialize associations efficiently.


## Live Coding

### Creating Tests for User Creation

When users sign up, we want to make sure that they are able to create a new account with a valid email and password. We will be using BCrypt to hash the password and store it in the database. We will also be using the has_secure_password method to authenticate the user.

Navigate to user_spec.rb and add the following tests:

#### spec/models/user_spec.rb

```ruby
require 'rails_helper'

RSpec.describe User, type: :model do
  it 'is valid with a username, password confirmation, and password' do
    user = build(:user, password: 'password', password_confirmation: 'password')
    expect(user).to be_valid
  end

  it 'is not valid without a username' do
    user = build(:user, username: nil)
    expect(user).not_to be_valid
  end
  
  it 'is not valid without an email' do
    user = build(:user, email: nil)
    expect(user).not_to be_valid
  end

  it 'is not valid without a first_name' do
    user = build(:user, first_name: nil)
    expect(user).not_to be_valid
  end
  
  it 'is not valid without a last_name' do
    user = build(:user, last_name: nil)
    expect(user).not_to be_valid
  end

  it 'hashes the password using BCrypt' do
    user = create(:user, password: 'password')

    expect(user.password_digest).not_to eq 'password'

    expect(BCrypt::Password.new(user.password_digest)).to be_truthy

    expect(BCrypt::Password.new(user.password_digest).is_password?('password')).to be true
  end
end
```

Let's include the validations for username, email, first_name, and last_name in the User model:

```ruby
class User < ApplicationRecord
  has_many :blogs
  has_one :profile
  after_create :create_profile

  validates :username, :email, :first_name, :last_name, presence: true
end
```

Now let’s run our tests:

```bash
bundle exec rspec
```

Remember, bundle exec refers to the bundle that was created when we ran bundle install. Just running rspec will only run the rspec that is installed globally on our machine. We want to run the rspec that is installed in our project.

```bash
..F....F.....

Failures:

  1) User is valid with a username, password confirmation, and password
     Failure/Error: user = build(:user, password: 'password', password_confirmation: 'password')
     
     NoMethodError:
       undefined method `password=' for #<User id: nil, first_name: "Clelia", last_name: "Ryan", email: "rogelio_gerhold@doyle-conroy.test", username: "adrien", created_at: nil, updated_at: nil>
     # ./spec/models/user_spec.rb:5:in `block (2 levels) in <main>'

  2) User hashes the password using BCrypt
     Failure/Error: user = create(:user, password: 'password')
     
     NoMethodError:
       undefined method `password=' for #<User id: nil, first_name: "Elfriede", last_name: "Frami", email: "yelena_hand@stanton.example", username: "sherman.price", created_at: nil, updated_at: nil>
     # ./spec/models/user_spec.rb:30:in `block (2 levels) in <main>'

Finished in 0.33075 seconds (files took 1.29 seconds to load)
13 examples, 2 failures

Failed examples:

rspec ./spec/models/user_spec.rb:4 # User is valid with a username, password confirmation, and password
rspec ./spec/models/user_spec.rb:29 # User hashes the password using BCrypt
```

To include attributes password and password_confirmation, we can use BCrypt to do just that. BCrypt will give us a method called has_secure_password that will allow us to authenticate the user. We can also use has_secure_password to validate the password and password_confirmation attributes.

#### gemfile

```ruby
gem 'bcrypt'
```

```bash
bundle install
```

#### app/models/user.rb

```ruby
class User < ApplicationRecord
  has_many :blogs
  has_one :profile
  after_create :create_profile

  validates :username, :email, :first_name, :last_name, presence: true
  has_secure_password
end
```

Let's run bundle exec rspec again:

```bash
.
.
.
  11) BlogsController POST #create with valid params renders a JSON response with the new blog
      Failure/Error: let(:user) { create(:user) }
      
      NoMethodError:
        undefined method `password_digest' for #<User id: nil, first_name: "Terry", last_name: "Crooks", email: "stevie@lockman.example", username: "donette", created_at: nil, updated_at: nil>
        Did you mean?  password
      # ./spec/requests/blogs_spec.rb:20:in `block (4 levels) in <main>'
      # ./spec/requests/blogs_spec.rb:21:in `block (4 levels) in <main>'
      # ./spec/requests/blogs_spec.rb:30:in `block (4 levels) in <main>'

Finished in 0.28806 seconds (files took 1.32 seconds to load)
13 examples, 11 failures
```

We get these errors since BCrypt is expecting a password_digest attribute. Let's create a migration to add a password_digest attribute to our User model:

```bash
rails generate migration AddPasswordDigestToUsers password_digest:string
```

```ruby
class AddPasswordDigestToUsers < ActiveRecord::Migration[7.0]
  def change
    add_column :users, :password_digest, :string
  end
end
```

```bash
rails db:migrate
```

Now running bundle exec rspec again:

```bash
.........FFF.

Failures:

  1) BlogsController GET #index returns a response with all the blogs
     Failure/Error: let(:blog) { create(:blog) } # Create a blog for the specific user
     
     ActiveRecord::RecordInvalid:
       Validation failed: Password can't be blank
     # ./spec/requests/blogs_spec.rb:4:in `block (2 levels) in <main>'
     # ./spec/requests/blogs_spec.rb:12:in `block (3 levels) in <main>'

  2) BlogsController POST #create with valid params creates a new Blog
     Failure/Error: let(:user) { create(:user) }
     
     ActiveRecord::RecordInvalid:
       Validation failed: Password can't be blank
     # ./spec/requests/blogs_spec.rb:20:in `block (4 levels) in <main>'
     # ./spec/requests/blogs_spec.rb:21:in `block (4 levels) in <main>'
     # ./spec/requests/blogs_spec.rb:25:in `block (5 levels) in <main>'
     # ./spec/requests/blogs_spec.rb:24:in `block (4 levels) in <main>'

  3) BlogsController POST #create with valid params renders a JSON response with the new blog
     Failure/Error: let(:user) { create(:user) }
     
     ActiveRecord::RecordInvalid:
       Validation failed: Password can't be blank
     # ./spec/requests/blogs_spec.rb:20:in `block (4 levels) in <main>'
     # ./spec/requests/blogs_spec.rb:21:in `block (4 levels) in <main>'
     # ./spec/requests/blogs_spec.rb:30:in `block (4 levels) in <main>'

Finished in 0.35337 seconds (files took 2.67 seconds to load)
13 examples, 3 failures

Failed examples:

rspec ./spec/requests/blogs_spec.rb:11 # BlogsController GET #index returns a response with all the blogs
rspec ./spec/requests/blogs_spec.rb:23 # BlogsController POST #create with valid params creates a new Blog
rspec ./spec/requests/blogs_spec.rb:29 # BlogsController POST #create with valid params renders a JSON response with the new blog
```

We get these errors because we are not passing in a password when we create a user. Let's update our factorybot to include a password and password_confirmation:

#### spec/factories/users.rb

```ruby
FactoryBot.define do
  factory :user do
    email { Faker::Internet.email }
    first_name { Faker::Name.first_name }
    last_name { Faker::Name.last_name }
    username { Faker::Internet.username }
    password { 'password' }
    password_confirmation { 'password' }
  end
end
```

Now running bundle exec rspec again:

```bash
.............

Finished in 0.37277 seconds (files took 1.37 seconds to load)
13 examples, 0 failures
```

Great! All of our tests are passing. Let's skip the tests for requests since that takes a bit more time to set up. We will be using Postman to test our requests.

### Creating a User via Postman

Let's add the create route to our routes.rb file for users

```ruby
Rails.application.routes.draw do
  resources :blogs
  resources :users, only: [:create]
end
```

Let's create a users controller and add a create action:

```bash
rails g controller users create
```

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
    params.permit(:username, :email, :first_name, :last_name, :password, :password_confirmation)
  end
end
```

Send a request using Postman and you should get a response with the user that was created:

```json
{
    "id": 3,
    "first_name": "John",
    "last_name": "Doe",
    "email": "test@test.com",
    "username": "test",
    "created_at": "2024-01-27T23:01:43.710Z",
    "updated_at": "2024-01-27T23:01:43.710Z",
    "password_digest": "$2a$12$v8bxkWeDzWR6ZBnKgh9O2.8jzeMD2kjPmp6FASFMBrGtg05O7cUcq"
}
```

Great, now bcrypt is hashing our password and storing it in the database.

### Login and JWT

Let's go ahead and add the following to our Gemfile:

```ruby
gem 'jwt'
```

```bash
bundle install
```

```ruby
Rails.application.routes.draw do
  post '/login', to: 'sessions#create'
  resources :blogs
  resources :users, only: [:create]
end
```

```bash
rails g controller sessions create
```

```ruby
class SessionsController < ApplicationController
  def create
    user = User.find_by(username: params[:username])
    if user&.authenticate(params[:password])
      token = jwt_encode(user_id: user.id)
      render json: { token: token }, status: :ok
    else
      render json: { error: "Unauthorized" }, status: :unauthorized
    end
  end

  private

  def jwt_encode(payload, exp = 24.hours.from_now)
    payload[:exp] = exp.to_i
    JWT.encode(payload, Rails.application.credentials.secret_key_base)
  end
end
```

The &. is called the safe navigation operator (also known as the "lonely operator").
Purpose:

It prevents an error when user is nil. Without it, if user were nil, calling authenticate on it would raise:

```bash
NoMethodError: undefined method `authenticate' for nil:NilClass
```

Go ahead and test this out using Postman; you should receive a JWT.

### Authenticating a user using JWT

Let's authenticate a user's JWT.

```ruby
class ApplicationController < ActionController::API
  def authenticate_request
    header = request.headers['Authorization']
    header = header.split(' ').last if header
    begin
      decoded = JWT.decode(header, Rails.application.credentials.secret_key_base).first
      @current_user = User.find(decoded['user_id'])
    rescue JWT::ExpiredSignature
      render json: { error: 'Token has expired' }, status: :unauthorized
    rescue JWT::DecodeError
      render json: { errors: 'Unauthorized' }, status: :unauthorized
    end
  end
end
```

```ruby
class BlogsController < ApplicationController
  before_action :authenticate_request
  .
  .
  .
```

Test for unauthorized and authorized users using Postman.

### Serialization with Blueprinter

Let’s go ahead and add the following to our Gemfile:

```ruby
gem 'blueprinter'
```

```bash
bundle install
```

```bash
rails g blueprinter:blueprint user
```

```ruby
# frozen_string_literal: true

class UserBlueprint < Blueprinter::Base
  identifier :id

  view :normal do
    fields :username, :email, :first_name, :last_name
  end

  view :blogs do 
    association :blogs, blueprint: BlogBlueprint
  end
end
```

```# frozen_string_literal: true``` is a magic comment, supported for the first time in Ruby 2.3, that tells Ruby that all string literals in the file are implicitly frozen, as if #freeze had been called on each of them. That is, if a string literal is defined in a file with this comment, and you call a method on that string which modifies it, such as <<, you'll get ```RuntimeError: can't modify frozen String```.

Freezing strings prevents bugs caused by accidentally modifying a string when you didn't intend to, and may improve performance.

```ruby
class UsersController < ApplicationController
  def create
    user = User.new(user_params)
    if user.save
      render json: UserBlueprint.render(user, view: :normal), status: :created
    else
      render json: user.errors, status: :unprocessable_entity
    end
  end

  private

  def user_params
    params.permit(:username, :email, :first_name, :last_name, :password, :password_confirmation)
  end
end
```

```bash
rails g blueprinter:blueprint blog
```

```ruby
# frozen_string_literal: true

class BlogBlueprint < Blueprinter::Base

  identifier :id

  fields :title, :content

  view :normal do
    
    association :user, blueprint: UserBlueprint, view: :normal
  end
end
```

```ruby
class BlogsController < ApplicationController
  before_action :authenticate_request

  def index
    blogs = Blog.all

    render json: BlogBlueprint.render(blogs, view: :normal)
  end

  def create 
    blog = Blog.new(blog_params)

    
    if blog.save
      render json: BlogBlueprint.render(blog, view: :normal), status: :created
    else
      render json: blog.errors, status: :unprocessable_entity
    end
  end

  private 

  def blog_params 
    params.permit(:title, :content, :user_id)
  end
end
```

Test this out using Postman.

# Any questions?