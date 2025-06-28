# Introduction to Ruby on Rails

## Introduction

-   We are going to go over the fundamentals of full stack web development
-   We are going to go over the fundamentals of Rails
-   We are going to start making an api

## Overview of Full Stack Web Development

### Understanding Frontend vs Backend Development
#### Frontend Development

  - Focuses on the user interface and experience.
  - Involves HTML, CSS, JavaScript, and frameworks like React, Angular, or Vue.js.
  - Emphasizes responsive design, performance optimization, and interactivity.

#### Backend Development

  - Deals with server, database, and application logic.
  - Involves languages like Ruby, Python, Java, PHP, and Node.js.
  - Centers on database management, server architecture, and API integration.

#### Integration

  - Frontend and backend must work seamlessly to deliver a complete web application.
  - Communication typically occurs via APIs, often following RESTful practices.

### Overview of Common Full Stack Technologies
#### Frontend Stack

  - HTML/CSS: Building blocks for web page structure and style.
  - JavaScript and Frameworks: For adding dynamics to web pages.
  - Libraries like jQuery and Bootstrap: Speed up development.

#### Backend Stack

  - Languages: Ruby, Python, PHP, Java, Node.js.
  - Frameworks: Ruby on Rails (Ruby), Django (Python), Express (Node.js).
  - Databases: SQL databases (MySQL, PostgreSQL) and NoSQL databases (MongoDB).
  - Server Management Tools: AWS, Heroku, etc.

#### DevOps and Version Control

  - Git for version control.
  - Docker, Jenkins, Kubernetes for continuous integration and deployment.

### Understanding the Internet and Web Servers
#### How the Internet Works

  - Based on client-server model where data is transmitted using TCP/IP protocol.
  - Clients (browsers) request data; servers respond with necessary data.

#### Web Servers

  - Store website files and respond to requests for these files.
  - Run on server software like Apache or Nginx.

#### Client-Server Interaction

  - Typing a URL in a browser sends a request to the server hosting the website.
  - The server processes this request and returns the appropriate response.

### IP Addresses and DNS
#### IP Addresses

  - Unique addresses assigned to every device on the Internet.
  - Essential for routing data packets to the correct destination.

#### DNS (Domain Name System)

  - Translates human-readable domain names (like google.com) into IP addresses.
  - Makes the internet user-friendly by allowing us to use domain names instead of IP addresses.

### Introduction to Web Hosting and Deployment
#### Web Hosting

  - Service that allows websites to be accessible via the internet.
  - Hosting providers offer server space for website files and ensure internet connectivity.

#### Deployment

  - Process of uploading and configuring website files on a hosting server.
  - Ensures that a website is accessible to users globally.

### HTTP Requests and Responses
#### HTTP Protocol

  - The foundation of data communication on the web.
  - Facilitates the request-response cycle between clients and servers.

#### HTTP Methods

  - GET: Retrieve data.
  - POST: Send data to create a new resource.
  - PUT: Update existing resources.
  - DELETE: Remove resources.
  - Others: PATCH, HEAD, OPTIONS.

#### HTTP Status Codes

  - Indicate the result of the HTTP request.
  - 200 Series: Success (e.g., 200 OK).
  - 300 Series: Redirection (e.g., 301 Moved Permanently).
  - 400 Series: Client errors (e.g., 404 Not Found).
  - 500 Series: Server errors (e.g., 500 Internal Server Error).

### REST and RESTful APIs
#### REST (Representational State Transfer)

  - An architectural style for designing networked applications.
  - Uses a stateless, client-server communication model.

  REST stands for Representational State Transfer. It's an architectural style for designing APIs. It's just a set of guidelines that developers can follow when building APIs. REST is based on the idea that everything is a resource and can be accessed using a unique identifier (which is called a URI). For example, a user is a resource and can be accessed using a URI like /users/1. REST is also stateless, which means that each request can be processed independently of the previous one. This makes REST APIs very scalable and easy to maintain.

#### URLs and Endpoints

  - URLs identify resources; endpoints determine how to interact with these resources. Example: GET /users/1 fetches user data for the user with ID 1.

#### RESTful Endpoints

  - Designed to perform specific actions using standard HTTP methods. E.g., POST /users to create a new user, PUT /users/1 to update user with ID 1.

## Ruby on Rails Introduction

### Installing Ruby on Rails

  - Install Ruby, then use `gem install rails` to install Rails.
  - Verify installation with `rails -v`.

### Creating a New Rails Project

  - Use `rails new project_name --api` for API-only applications.
  - This sets up a Rails project optimized for building APIs.

### Rails Directory Structure

  - Key directories: app, bin, config, db, lib, log, public, test, tmp, vendor
  - app/models: Contains models for handling data.
  - config/routes.rb: Defines URL routes.

## Codelabs Learning Assistant

> [Codelabs Learning Assistant Demo](https://chatgpt.com/g/g-68484cbcb348819181c3f4137b0b7c49-codelabs-learning-assistant)
>
> -   Demonstrate for students how to use the Codelabs Learning Assistant for creative exploration: "Can you summarize in two sentences what the 'app' folder is for in a Rails project?"

### Model, View, Controller (MVC) Pattern in Rails

  - Rails follows the MVC architecture, separating data (Model), user interface (View), and control logic (Controller).
  - Models: Interact with the database and manage application data.
  - Views: Present data to the user in a specified format.
  - Controllers: Process user requests, interact with models, and render views.

### Rails CLI and Generators

  - Rails CLI is used to create models, controllers, and more. Example: `rails generate model User name:string email:string` creates a User model with name and email fields.

### Migrations

  - Rails uses migrations to modify the database schema. Example: `rails generate migration AddAgeToUsers age:integer` adds an age column to the Users table.

### Interacting with the Database

  - Rails ORM (Object-Relational Mapping) allows interaction with the database using Ruby code. Example: `User.create(name: 'Alice', email: 'alice@example.com')` adds a new user to the database.

### Validations and Associations

  - Validations ensure data integrity (e.g., validates :name, presence: true).
  - Associations define relationships between models (e.g., a User has_many :posts).

### Rails Console

  - An interactive shell for experimenting with Rails applications.
  - Accessible via `rails console`.

## Codelabs Learning Assistant

## Live Coding

In this lesson, we delve into the creation and management of multiple tables in Ruby on Rails and explore how to define relationships between different data models.

## Beta Blogs

### Setup

Initialize a New Rails Project:

```bash
rails new beta_blogs_api --api
```

This command creates a new Rails API project named "beta_blogs_api".

Navigate to Project Directory:

```bash
cd beta_blogs_api
```

### Creating Models

Create user model:

```bash
rails g model User first_name:string last_name:string email:string username:string
```

This command generates a User model with first_name, last_name, email, and username columns. It will create the following files

  - app/models/user.rb
  - db/migrate/time_stamp_create_users.rb
  - more surrounding testing

Execute pending migrations:

```bash
rails db:migrate
```

Let's create a user record. Let's navigate to the Rails console:

```bash
rails console
```

The rails console is an interactive shell for accessing the Rails applications. It allows us to interact with the database.

Create a user record:

```bash
user = User.create(first_name: 'John', last_name: 'Doe', email: 'johndoe123@email.com', username: 'johndoe123')
```

Create blog model:

```bash
rails g model Blog title:string content:text user:references
```

Create a profile model:

```bash
rails g model Profile user:references bio:text
```

### Creating Associations in Rails
Creating One-to-One Relationship:

A one to one relationship is when one record in a table is associated with one record in another table. For example, a user has one profile.

```ruby
class User < ApplicationRecord
  has_one :profile
end
```

```ruby
class Profile < ApplicationRecord
  belongs_to :user
end
```

Let's create a user with a profile:

```bash
user = User.first
profile = Profile.create(user: user, bio: "I'm a software engineer.")
user.profile
```

After_create Callbacks

Let's go ahead and create a profile for every user that is created. We can do this using an after_create callback.

```ruby
class User < ApplicationRecord
  has_one :profile
  after_create :create_profile
end
```

`after_create` is a callback that is called after a record is created.

`create_profile` is a pre-defined instance method in which was defined once we established the relationship between the user and profile model.

By executing the `create_profile` method, we are creating a profile for every user that is created.

```bash
user = User.create(first_name: "Jane", last_name: "Doe", email: "janedoe123@gmail.com", username: "janedoe123")
user.profile
```

### One-to-Many Associations
Creating One-to-Many Relationship:

A one to many relationship is when one record in a table is associated with many records in another table. For example, a user has many blogs. And a blog belongs to a user.

```ruby
class User < ApplicationRecord
  has_many :blogs
  has_one :profile
  after_create :create_profile
end
```

```ruby
class Blog < ApplicationRecord
  belongs_to :user
end
```

Let's create a user with multiple blogs:

```bash
user = User.first
blog1 = Blog.create(user: user, title: "My First Blog", content: "This is my first blog.")
blog2 = Blog.create(user: user, title: "My Second Blog", content: "This is my second blog.")
user.blogs
```

### Model Validations

Model validations are used to ensure that only valid data is saved into the database. For example, it may be important to your application to ensure that every user provides a valid email address and mailing address. Model validations are the best way to ensure that only valid data is saved into your database.

Adding Validations to User Model:

```ruby
class User < ApplicationRecord
  has_many :blogs
  has_one :profile
  after_create :create_profile

  validates :first_name, presence: true
  validates :last_name, presence: true
  validates :email, presence: true
  validates :username, presence: true
end
```

> [Codelabs Learning Assistant Demo](https://chatgpt.com/g/g-68484cbcb348819181c3f4137b0b7c49-codelabs-learning-assistant)
>
> -   Demonstrate for students how to use the Codelabs Learning Assistant for creative exploration: "How do you add a validation to make sure an email is unique in Rails? Can you show me a code example?"

Let's try creating a user without any information:

```bash
User.create
```

This will return an error because the first_name, last_name, email, and username are required.

Adding Validations to Blog Model:

Let's go ahead and require that the title and content are present in the blog model.

```ruby
class Blog < ApplicationRecord
  belongs_to :user

  validates :title, presence: true
  validates :content, presence: true
end
```

Let's try creating a blog without any information:

```bash
Blog.create
```

This will return an error because the title and content are required.

# Any questions?
