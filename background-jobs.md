# Background jobs and task scheduling using tools like Sidekiq

## Introduction

- We are going to discuss background jobs and how we can implement them with Sidekiq.

## Live Coding

Since we are limited with our deployment options, we will be using a local setup for this tutorial.

Create a new branch called `sidekiq scheduler` and checkout to it.

We will create a monthly report model and a worker to generate the report.

```bash
rails generate model MonthlySummary user:references month:date total_likes:integer
```

```bash
rails db:migrate
```

Add the association to users:

```ruby
class User < ApplicationRecord
  has_many :blogs
  has_one :profile
  has_many :likes
  has_many :monthly_summaries
  .
  .
  .
```

Setup Sidekiq

Make sure to have Redis installed on your machine. And the redis server is running.

```bash
redis-server
```

Add the following gems to your Gemfile.

```ruby
gem 'sidekiq'
gem 'sidekiq-scheduler'
```

Install the gems.

```bash
bundle install
```

To create a job to generate the monthly report, run the following command.

```bash
rails generate sidekiq:job MonthlyReportSummary
```

```ruby
class MonthlyReportSummaryJob
  include Sidekiq::Job

  def perform(*args)
    last_month = Date.today.last_month
    User.find_each do |user|
      total_likes = 0

      user.blogs.where(created_at: last_month.beginning_of_month..last_month.end_of_month).each do |blog|
        total_likes += blog.likes.count
      end


      MonthlySummary.create(user: user, month: last_month.beginning_of_month) do |summary|
        summary.total_likes = total_likes
      end

      puts "Monthly report summary created for #{user.username} with #{total_likes} likes."
    end
  end
end
```

Now let's schedule this job to run every month.

We can schedule the job to run at regular intervals by adding the following line to the `config/sidekiq.yml` file. Be sure to create this file.

Visit the following url to create a scheduled cron.

```yaml
:scheduler:
  :schedule:
    monthly_report_summary:
      cron: '0 0 1 * *' # Run at midnight on the first day of every month
      class: 'MonthlyReportSummaryJob'
```

To explain the cron expression, the first 0 is for the minute, the second 0 is for the hour, the 1 is for the day of the month, and the \* is for the month.

Here we are saying that we want to run the `MonthlyReportSummaryJob` at midnight on the first day of every month.

Now, let's start the sidekiq server.

```bash
bundle exec sidekiq
```

Let's seed some data to test the scheduler.

```ruby
User.find_each do |user|
  rand(1..5).times do
    blog = user.blogs.create!(
      title: 'Title',
      content: 'Content',
      created_at: 1.month.ago
    )

    # give random likes to the blog
    rand(1..10).times do |i|

      liker = User.offset(rand(User.count)).first

      liker.likes.create!(likeable: blog)
    end
  end
end
```

```bash
rails db:seed
```

The problem with this approach is that we have to wait for the next month to see the report.

Instead of waiting, we can perform the job manually in the console.

```bash
MonthlyReportSummaryJob.perform_async
```

You should see something similar to following output in the sidekiq terminal.

```bash
Monthly report summary created for johndoe123 with 17 likes.
Monthly report summary created for  with 35 likes.
Monthly report summary created for test with 26 likes.
Monthly report summary created for test with 15 likes.
Monthly report summary created for jimmy123 with 9 likes.
Monthly report summary created for username with 25 likes.
Monthly report summary created for test2 with 9 likes.
```

# Any questions?
