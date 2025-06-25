# Ruby Gems and CLI Part 2

## Introduction

-   We're going to continue to explore program structure making another CLI
-   bcrypt and password hashing

# Project Walkthrough

## Step 1

-   Create a new file called user.rb in your lib folder
-   Create a new class called User

### lib/user.rb

```ruby
class User
	@@users = []

	attr_accessor :id, :username, :password

	def initialize(username, password)
		@username = username
		@password = password
		@id = @@users.length + 1

		@@users << self
	end

	# Return all Users from array
	def self.all
		@@users
	end

	# Populate the Users array
	def self.seed
		users = [{username: "joe", password: "password"}]

		users.each do |user|
			User.new(user[:username], user[:password])
		end
	end

    # Find a specific User by id
	def self.find(id)
		self.all.find { |user| user.id == id }
	end
end
```

## Codelabs Learning Assistant

> [Codelabs Learning Assistant Demo](https://chatgpt.com/g/g-68484cbcb348819181c3f4137b0b7c49-codelabs-learning-assistant)
>
> -   Copy the User class above and paste it into the assistant.
> -   Ask: Can you break down what each method in this class does?
> -   Discuss the assistant's breakdown with the class.

## Step 2

-   Add bcrypt to your Gemfile

```ruby
gem 'bcyrypt'
```

-   Run bundle install

```
bundle install
```

## Codelabs Learning Assistant

> [Codelabs Learning Assistant Demo](https://chatgpt.com/g/g-68484cbcb348819181c3f4137b0b7c49-codelabs-learning-assistant)
>
> -   Ask the assistant: How does bcrypt hash a password in Ruby?
> -   Show the response and relate it to the next code example.

## Step 3

-   Create a new file under lib called auth.rb

```ruby
require "bcrypt"

module Auth
	def self.create_hash_digest(password)
		BCrypt::Password.create(password)
	end

	def self.verify_hash_digest(hashed_password, plain_password)
	# Password.new returns an object that can be compared to the plain password
		BCrypt::Password.new(hashed_password) == plain_password
	end

	def self.authenticate_user(username, password, list_of_users)
		list_of_users.each do |user_record|
			found_username = user_record.username == username

            next unless found_username

     	    hash_password = user_record.password
            match_password = self.verify_hash_digest(hash_password, password)

			return user_record if found_username && match_password
		end
		return nil
	end
end
```

## Codelabs Learning Assistant

> [Codelabs Learning Assistant Demo](https://chatgpt.com/g/g-68484cbcb348819181c3f4137b0b7c49-codelabs-learning-assistant)
>
> -   Ask the assistant: How does the Auth module verify a user's password?
> -   Use the assistant's answer to clarify any confusion about authentication logic.

## Step 4

-   Go back to your user.rb file
-   Require the Auth module into your User Class
-   When you create a new user, you want to hash the password

```ruby
require_relative "auth.rb"

class User
	@@users = []

	attr_accessor :id, :username, :password

	def initialize(username, password)
		# . . .
		@password = Auth.create_hash_digest(password)
		# . . .
	end
end
```

-   Require the user and auth in main file

```ruby
require_relative 'lib/user'
require_relative 'lib/auth'
```

## Step 5 (Sign In)

-   Seed the users when you run the program
-   Ensure they are authenticated before they can access the menu by creating a sign_in method and placing it after the greet method call
-   sign_in should return the user object if they are authenticated otherwise it should send a message to the user that they are not authenticated

```ruby
class CLI
	def run
		User.seed
		system("clear")
		greet
		sign_in
		menu until menu == "exit"
		goodbye
	end

	def sign_in
		is_authenticated = false

		until is_authenticated
			puts "Please enter your username: "

			username = gets.chomp

			puts "Please enter your password: "

			password = gets.chomp

			is_authenticated = Auth.authenticate_user(username, password, User.all)

			if is_authenticated
				puts "Welcome, #{username}!"
			else
				puts "Invalid credentials. Please try again."
			end
		end
	end
end
```

## Codelabs Learning Assistant

> [Codelabs Learning Assistant Demo](https://chatgpt.com/g/g-68484cbcb348819181c3f4137b0b7c49-codelabs-learning-assistant)
>
> -   Ask the assistant: Can you explain the sign_in method in this CLI class?
> -   Use the assistant's explanation to reinforce the sign-in logic.

## Step 6 (Sign Up)

In the cli.rb file:

-   Create the sign_up method

```ruby
def sign_up
    puts "Sign Up\n\n"

    puts "Please enter a username: "

    username = gets.chomp

    puts "Please enter a password: "

    password = gets.chomp

    if username == "" || password == ""
        puts "Invalid credentials. Please try again."

        sign_up
        return nil
    end

    User.new(username, password)

    puts "Welcome, #{username}! Please sign in to continue."

    sign_in
end
```

-   Create an enter_credentials method

```ruby
def run
    User.seed
    system("clear")
    greet
    enter_credentials
    while menu != "exit"
    end
    goodbye
end

def enter_credentials
        puts "Do You Have An Account? Type 'Y' to Sign In or 'N' to Sign Up: "

        has_account = gets.chomp.downcase == "y"

        if has_account
            sign_in
        else
            sign_up
        end
end
```

-   Refactor the sign_in and goodbye methods

```ruby
def sign_in
    is_authenticated = false

    puts "Sign In\n\n"

    until is_authenticated
        # . . .

        if is_authenticated
            @current_user = is_authenticated
            puts "Welcome, #{@current_user.username}!"
        else
            puts "Invalid credentials. Please try again."
        end
    end
end

#. . .

def goodbye
    puts "Goodbye, #{@current_user.username}! :("
end
```

# Any questions?
