# Testing with RSpec

## Introduction

-   We are going to discuss the value of testing
-   We are going to discuss different kinds of testing
-   We are going to add rspec tests to our api

## Testing Discussion

> [Codelabs Learning Assistant Demo](https://chatgpt.com/g/g-68484cbcb348819181c3f4137b0b7c49-codelabs-learning-assistant)
>
> - Ask the Codelabs Learning Assistant: “Can you give me a real world example that explains why testing is important in software development?”


  - Testing is essential for reliable and maintainable Rails applications.
  - RSpec enhances testing capability in Rails.
  - Writing tests before code (TDD) leads to more reliable software development.
  - Different types of tests cover various aspects of the application.
  - Unit and controller tests are key in Rails to ensure models and actions work as intended.
  - Refactoring with the support of tests ensures code improvements don't introduce new issues.

### About Testing
  - Testing verifies code works as expected.
  - Benefits: Finds bugs, improves code quality, supports refactoring, documents code, aids collaboration, and enhances product quality.

### Test Driven Development (TDD)
  - Write tests before code.
  - Process: Write a test, run and watch it fail, write code to pass the test, refactor.

### Types of Testing
  - Unit Testing: Tests individual units of code.
  - Integration Testing: Tests how different units work together.
  - Functional Testing: Tests the application's functionality.
  - Acceptance Testing: Tests the application against acceptance criteria.
  - Regression Testing: Ensures new code doesn't break existing code.
  - Performance Testing: Tests application performance.
  - Security Testing: Tests application security.

### Testing in Ruby on Rails
  - Rails ships with Minitest.
  - RSpec is a more powerful alternative.
  - Use rails new rails-testing -T --api to generate Rails app with RSpec.
  - Add rspec-rails and factory_bot_rails gems for testing.
  - factory_bot_rails is a gem that allows you to create test data.
  - Faker is a gem that generates fake data for testing.

## Live Coding

### Setup

To remove all mini-test files, run the following command:

```bash
rm -rf test/
```

Or simply delete the test directory.

Let's create our testing environment by including gems for testing in the Gemfile.

```ruby
group :development, :test do
  # See https://guides.rubyonrails.org/debugging_rails_applications.html#debugging-with-the-debug-gem
  gem "debug", platforms: %i[ mri mingw x64_mingw ]
  gem 'rspec-rails'
  gem 'factory_bot_rails'
  gem 'faker'
end
```

  - What does group :development, :test mean?
  - What is and where is the production group?

```bash
bundle install
```

Next, run the following command to generate the RSpec configuration files:

```bash
rails generate rspec:install
```

Be sure to include

```ruby
require 'faker'
.
.
.
```

in the rails_helper.rb file.

### Unit Testing Models

Unit tests verify individual units of code.

Let's create a unit test for the Blog model.

Keep in mind, we are not going to be using factory bot or faker for this demonstration.

```bash
rails generate rspec:model Blog
```

This will generate a file in the spec/models directory called blog_spec.rb and a file in the spec/factories directory called blogs.rb.

#### spec/models/blog_spec.rb

```ruby
require 'rails_helper'

RSpec.describe Blog, type: :model do
  pending "add some examples to (or delete) #{__FILE__}"
end
```

Let's test the Blog model's validations.

#### spec/models/blog_spec.rb

```ruby
# spec/models/blog_spec.rb
require 'rails_helper'

RSpec.describe Blog, type: :model do

  context 'Validation tests' do
    it 'is not valid without a title' do
      user = User.create(first_name: "john", last_name: "doe", username: "johndoe", email: "johndoe123@gmail.com")
      blog = Blog.new(content: 'Sample content', user: user)
      expect(blog).to_not be_valid
      expect(blog.errors[:title]).to include("can't be blank") # Optional
    end

    it 'is not valid without content' do
      user = User.create(first_name: "john", last_name: "doe", username: "johndoe", email: "johndoe123@gmail.com")

      blog = Blog.new(title: 'Sample Title', user: user)
      expect(blog).to_not be_valid
      expect(blog.errors[:content]).to include("can't be blank") # Optional
    end
  end
end
```

Run the test:

```bash
bundle exec rspec spec/models/blog_spec.rb
```

or

```bash
bundle exec rspec 
```

We might be expecting failure tests and if so we should see the following output:

```bash
.FF

Failures:

  1) Blog is not valid without a title
     Failure/Error: expect(blog).to_not be_valid
       expected #&lt;Blog id: nil, title: nil, content: nil, user_id: nil, created_at: nil, updated_at: nil&gt; not to be valid
     # ./spec/models/blog_spec.rb:10:in `block (2 levels) in &lt;top (required)&gt;&#39;

  2) Blog is not valid without content
     Failure/Error: expect(blog).to_not be_valid
       expected #&lt;Blog id: nil, title: nil, content: nil, user_id: nil, created_at: nil, updated_at: nil&gt; not to be valid
     # ./spec/models/blog_spec.rb:15:in `block (2 levels) in &lt;top (required)&gt;&#39;

Finished in 0.0317 seconds (files took 1.49 seconds to load)
3 examples, 2 failures

Failed examples:

rspec ./spec/models/blog_spec.rb:8 # Blog is not valid without a title
rspec ./spec/models/blog_spec.rb:13 # Blog is not valid without content
```

Let's pass the tests by adding validations to the Blog model.

```ruby
class Blog < ApplicationRecord
  has_and_belongs_to_many :categories
  belongs_to :user
  validates :title, :content, presence: true
end
```

```bash
...
Finished in 0.03475 seconds (files took 1.76 seconds to load)
3 examples, 0 failures
```

### Testing Requests

Index Action

Let's create a unit test for the BlogsController.

```bash
rails generate rspec:request Blogs
```

This will generate a file in the spec/requests directory called blogs_spec.rb.

#### spec/requests/blogs_spec.rb

```ruby
require 'rails_helper'

RSpec.describe "Blogs", type: :request do

end
```

Let's test the index action.

First let's setup our test data using the factory_bot_rails gem and Faker gem.

#### spec/factories/blogs.rb

```ruby
FactoryBot.define do
  factory :blog do
    title { Faker::Lorem.sentence }
    content { Faker::Lorem.paragraph }
    user
  end
end
```

Create a user factory:

#### spec/factories/users.rb

```ruby
FactoryBot.define do
  factory :user do
    email { Faker::Internet.email }
    first_name { Faker::Name.first_name }
    last_name { Faker::Name.last_name }
    username { Faker::Internet.username }
  end
end
```

To use factory-bot, we need to add the following to our rails_helper.rb file:

```ruby
.
.
.
RSpec.configure do |config|
  config.include FactoryBot::Syntax::Methods
  .
  .
  .
end
```

Let's test the index action using the factory data.

#### spec/controllers/blogs_controller_spec.rb

```ruby
require 'rails_helper'

RSpec.describe "Blogs", type: :request do
  describe "GET /index" do
  let(:blog) { create(:blog) }
  
    before do 
      blog
      get '/blogs'
    end

    it "returns http success" do
      expect(response).to have_http_status(:success)
    end

    it 'returns a list of blogs' do
      expect(response.body).to eq(Blog.all.to_json)
    end
  end
end
```

The let method in RSpec is lazy-evaluated. This means that the code inside let (create(:user)) won't actually run until user is explicitly called in the test. If you didn't call user before get :index, no user would be created, and your index action would be returning an empty array (assuming the database is empty to start with).

Let's pass the tests by adding the index action to the BlogsController if we haven't already.

#### config/routes.rb

```ruby
Rails.application.routes.draw do
  resources :blogs
end
```

This will give us the following routes:

```
| Prefix | Verb   | URI Pattern          | Controller#Action |
| ------ | -------| -------------------- | ----------------- |
| blogs  | GET    | /blogs(.:format)     | blogs#index       |
|        | POST   | /blogs(.:format)     | blogs#create      |
| blog   | GET    | /blogs/:id(.:format) | blogs#show        |
|        | PATCH  | /blogs/:id(.:format) | blogs#update      |
|        | PUT    | /blogs/:id(.:format) | blogs#update      |
|        | DELETE | /blogs/:id(.:format) | blogs#destroy     |
```

#### app/controllers/blogs_controller.rb

```ruby
class BlogsController < ApplicationController
  def index
    blogs = Blog.all

    render json: blogs
  end
end
```

```bash
rspec
```

Create Action

Let's test the create action by adding onto the blogs_controller_spec.rb file.

#### spec/controllers/blogs_controller_spec.rb

```ruby
require "rails_helper"

RSpec.describe "Blogs", type: :request do
# .
# .
# .
  describe "POST /create" do
  let (:user) { create(:user) }

    context 'when the blog is valid' do
      before do
        post '/blogs', params: {title: 'Test Title', content: 'Test Content', user_id: user.id }
      end

      it 'returns http created' do
        expect(response).to have_http_status(:created)
      end

      it 'returns the created blog' do
        expect(response.body).to eq(Blog.last.to_json)
      end

    end

    context 'when the blog is invalid' do
      before do
        post '/blogs', params: {  title: 'Test Title', content: 'Test Content', user_id: nil } 
      end

      it 'returns http unprocessable entity' do
        expect(response).to have_http_status(:unprocessable_entity)
      end

      it 'returns the validation errors' do
        expect(JSON.parse(response.body)).to eq({"user"=>["must exist"]})
      end
    end
  end
end
```

Let's pass this:

#### app/controllers/blogs_controller.rb

```ruby
class BlogsController < ApplicationController
# .
# .
# .
  def create 
    blog = Blog.new(blog_params)

    
    if blog.save
      render json: blog, status: :created
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

```bash
rspec
```

# Any questions?