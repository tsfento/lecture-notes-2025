# Refactoring with Services and PORO Pattern

## Introduction

- We are going to refactor our app to use services.

## Live Coding

Let's create a service for creating a user. We will extract the logic of creating a user into a service. Create a folder called `services` in app. Let's create a new file in our services folder called `user_service.rb`. Restart the service if you are running the server.

Include the following code in the `user_service.rb` file:

```ruby
module UserService
  module Base
    def self.create_user(params)
      user = User.new(params)
      if user.save
        user
      else
        user.errors
      end
    end
  end
end
```

In our users controller, let's add the following:

```ruby
class UsersController < ApplicationController
  def create
    user = UserService::Base.create_user(user_params)
    if user.valid?
      render json: user, status: :created
    else
      render json: user, status: :unprocessable_entity
    end
  end
  .
  .
  .
end
```

Standardizing the return of a service method is a good practice. Let's create a file called `service_contract.rb` in services. Include the following code:

```ruby
# frozen_string_literal: true

require 'ostruct'

# Standardizes what a service method should always return
module ServiceContract
  def self.success(payload)
    OpenStruct.new({ success?: true, payload: payload, errors: nil })
  end

  def self.error(errors)
    OpenStruct.new({ success?: false, payload: nil, errors: errors })
  end
end
```

`OpenStruct` is used here so we can return a neat little object that looks like it has real attributes (`success?`, `payload`, `errors`) without writing a whole class. It keeps the example short and readable. But in real apps, many teams prefer to define a dedicated `ServiceResult` class instead. That way, your return values are more explicit, easier to test, and safer against accidental misuse.

Let's update our `user_service.rb` file to use the `ServiceContract`:

```ruby
module UserService
  module Base
    def self.create_user(params)
      user = User.new(params)

      begin
        # are there any db/model errors?
        user.save!
      rescue ActiveRecord::RecordInvalid => exception
        # return an error instance
        return ServiceContract.error(user.errors.full_messages) unless user.valid?
      end

      ServiceContract.success(user)
    end
  end
end
```

Notice that in our redefinition of the `create_user` method, we are using the `ServiceContract` module to return the success or failure of the operation. This makes it easier to understand the success or failure of the operation.

Let's update our users controller to reflect these changes

```ruby
class UsersController < ApplicationController
  def create
    result = UserService::Base.create_user(user_params)
    if result.success?
      render json: UserBlueprint.render(result.payload, view: :normal), status: :created
    else
      render json: result.errors, status: :unprocessable_entity
    end
  end

  private

  def user_params
    params.permit(:username, :email, :first_name, :last_name, :password, :password_confirmation)
  end
end
```

Great! We have successfully implemented the PORO pattern in our Rails application. This pattern is useful for extracting business logic from the controller and keeping the controller thin. It also makes the code more readable and maintainable.

Let's dig deeper by adding a series of methods that will allow us to redefine response methods.

In our application controller file, let's add the following:

```ruby
def render_error(errors:, status: :internal_server_error)
  render json: {
    success: false,
    errors: errors,
    status: status
  }, status: status
end

def render_success(payload:, status: :ok)
  render json: {
    success: true,
    payload: payload
  }, status: status
end
```

This will allow us to redefine the response methods in our controllers. It's easier to read and understand the success or failure of the operation.

In our users_controller, let's update the create method to reflect these changes:

```ruby
def create
  result = UserService::Base.create_user(user_params)
  if result.success?
    render_success(payload: UserBlueprint.render_as_hash(result.payload, view: :normal), status: :created)
  else
    render_error(errors: result.errors, status: :unprocessable_entity)
  end
end
```

Try testing with postman.

Make sure to use `render_as_hash` of blueprinter to render the user as a hash. This will allow us to render the user as a hash and as a JSON object.
Refactoring the Blogs Controller

Let's add pagination to our index action in the blogs controller.

```ruby
def index
  page = params.fetch(:page, 1).to_i
  per_page = params.fetch(:per_page, 10).to_i

  offset = (page - 1) * per_page

  blogs = Blog.offset(offset).limit(per_page)

  render_success(payload: BlogBlueprint.render_as_hash(blogs, view: :normal, current_user: @current_user), status: :ok)
end
```

Let's extract this into a service called `blog_service.rb`.

```ruby
module BlogService
  module Base
    def self.filter(params)
      # fetch means that if the key is not found, it will return the default value
      page = params.fetch(:page, 1).to_i
      per_page = params.fetch(:per_page, 10).to_i

      begin
        # Calculate the offset based on the current page
        offset = (page - 1) * per_page

        # Fetch the subset of blogs based on pagination parameters
        blogs = Blog.offset(offset).limit(per_page)
      rescue ActiveRecord::RecordInvalid => exception
        return ServiceContract.error(exception.record.errors.full_messages) unless exception.record.valid?
      end

      ServiceContract.success(blogs)
    end
  end
end
```

Then update the blogs controller to use the blog service.

```ruby
def index
  result = BlogService::Base.filter(params)
  if result.success?
    render_success(payload: BlogBlueprint.render_as_hash(result.payload, view: :normal, current_user: @current_user), status: :ok)
  else
    render_error(errors: result.errors, status: :unprocessable_entity)
  end
end

# Make sure you update your permitted params
def blog_params
  params.permit(:title, :content, :cover_image, :page, :per_page)
end
```

This is much easier to read and understand.

Try sending a request to `localhost:3000/blogs?page=1&per_page=4`. You should see the first 4 blogs in the response.

# Any questions?
