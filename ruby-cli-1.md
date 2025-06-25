# Ruby Gems and CLI Part 1

## Introduction

-   Make a CLI to broaden what we discussed with Classes and to more illustrate how to put a program together and reference scripts
-   Start to explore gems and how they can enhance your programs

## What is a CLI?

### Can anyone explain what a CLI is?

-   A CLI is a Command Line Interface. It is a program that allows you to interact with your computer using the command line.

### What are some examples of a CLI?

-   An example would be the Angular CLI. It allows you to create a new angular project, generate components, generate services, etc.
-   Bash, Powershell, or your terminal of choice

# Project Walkthrough

## Create the Project

We will be using the [Oscar Winning Films](https://www.scrapethissite.com/pages/ajax-javascript) website to build our CLI application. Our application will allow the user to enter a year and see the Oscar winning films for that year.

In VS Code, create a new folder called oscar_award_winning_movies.

Create a Lib folder, and a new file in the lib folder called cli.rb.

Create a new class called CLI and a simple method called run that prints "Hello, World!" to start.

### lib/cli.rb

```ruby
class CLI
  def run
    puts "Hello, World!"
  end
end
```

## Run the CLI

Create a file called main.rb and require the cli.rb file outside of lib.

### main.rb

```ruby
require_relative 'lib/cli.rb'

CLI.new.run
```

Execute the program by running `ruby main.rb` in your terminal.

## Greet the user and say goodbye

In your cli.rb file, create a new method called greet that prints "Welcome to the Oscar Winning Films CLI!"

### lib/cli.rb

```ruby
class CLI
  def run
    system("clear")
    greet
  end

  def greet
    puts "Welcome to the Oscar Winning Films CLI!"
  end
end
```

Run the program to ensure you have no errors.

Create another method called goodbye that prints "Goodbye!"

### lib/cli.rb

```ruby
class CLI
  def run
    system("clear")
    greet
    goodbye
  end

  def greet
    puts "Welcome to the Oscar Winning Films CLI!"
  end

  def goodbye
    puts "Goodbye!"
  end
end
```

Run the program to ensure you have no errors.

## Create the menu

In your cli.rb file, create a new method called menu that prints "Please enter a number to see the Oscar winning film for that year:"

### lib/cli.rb

```ruby
class CLI
  def run
    puts "Hello, World!"
    greet
    menu
    goodbye
  end

  def greet
    puts "Welcome to the Oscar Winning Films CLI!"
  end

  def goodbye
    puts "Goodbye!"
  end

  def menu
    puts "\n Please enter a number to see the Oscar winning films for that year: \n\n"

    input_year = gets.chomp

    input_year
  end
end
```

## Can anyone explain gets.chomp?

-   gets gets the user input
-   Adding chomp removes the automatic newline from user input

We need to add a loop to our run method.

### lib/cli.rb

```ruby
class CLI
  def run
    system("clear")
    greet
    while menu != "exit"
    end
    goodbye
  end

  # . . .
end
```

Here we are saying, while the menu method does not return "exit", keep running the program.

## Find the films by year

In your cli.rb file, create a new method called find_films_by_year(user_input) that prints "You entered #{year}!"

###lib/cli.rb

```ruby
class CLI
  # . . .

  def menu
    puts "\n Please enter a number to see the Oscar winning films for that year: \n\n"
    input_year = gets.chomp
    find_films_by_year(input_year)
    input_year
  end

  def find_films_by_year(year)
    puts "\n Let me find the Oscar winning films for #{year}... \n\n"
  end
end
```

Run the program to ensure you have no errors.

## Use the oscar winning films API

Create a new file in the lib folder called api.rb

Create a new class called API and a simple method called find_films_by_year(year) that prints "You entered #{year}!"

### lib/api.rb

```ruby
class API
  def self.find_films_by_year(year)
  end
end
```

### lib/cli.rb

```ruby
require_relative "api.rb"
.
.
.
```

Call the find_films_by_year method from the find_films_by_year method in the cli.rb file

```ruby
class CLI
  # . . .

  def find_films_by_year(year)
    puts "\n Let me find the Oscar winning films for #{year}... \n\n"
    API.find_films_by_year(year)
  end
end
```

Run the project to ensure you have no errors.

## Can anyone tell me what gems are?

-   Ruby gems are packages of code that you can use in your Ruby projects.

## Add gems to the project

Create a Gemfile in the root (outside of lib) file and add "nokogiri", "open-uri" and "byebug". Then run bundle install in your shell.

In your Gemfile file, add "nokogiri", "open-uri" and "byebug".

```ruby
source "https://rubygems.org"
gem "nokogiri"
gem 'open-uri'
```

Then run bundle install in your shell.

In your api.rb file, require

```ruby
require "nokogiri"
require "open-uri"
require "json"
```

## Create the BASE_URL

In your api.rb file, create a constant called BASE_URL and set it equal to https://www.scrapethissite.com/pages/ajax-javascript/?ajax=true&year= at the top of your class.

Create a constant called BASE_URL and set it equal to https://www.scrapethissite.com/pages/ajax-javascript/?ajax=true&year= at the top of your class

```ruby
BASE_URL = "https://www.scrapethissite.com/pages/ajax-javascript/?ajax=true&year="
```

Use NokoGiri to scrape the data from the website and parse the data into a JSON object

```ruby
def self.find_films_by_year(year)
  doc = Nokogiri::HTML(URI.open(BASE_URL + year))
  scraped_movies = JSON.parse(doc.text)
end
```

Add a conditional to check if the scraped movies are empty. If they are, print "Sorry, I couldn't find any Oscar winning films for #{year}."

```ruby
def self.find_films_by_year(year)
  doc = Nokogiri::HTML(URI.open(BASE_URL + year))
  scraped_movies = JSON.parse(doc.text)

  if scraped_movies.empty?
    puts "Sorry, I couldn't find any Oscar winning films for #{year}."
  end

  scraped_movies.each do |movie|
    puts "| #{movie["title"]} | #{movie["year"]} | #{movie["nominations"]} | #{movie["best_picture"]}"
  end
end
```

Try to search for 2010 and see if you get any results.

# Any questions?
