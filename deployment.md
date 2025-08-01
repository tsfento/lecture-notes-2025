# Deployment

## Introduction

- We are going to be deploying both repos to hosting services

## Live Coding

Setting up the API to use Render

Since Render, and most services really, require postgresql as the database, we will need to include pg in our Gemfile.

```ruby
source "https://rubygems.org"

ruby "3.1.2"

gem "rails", "~> 7.1.3"

# Use sqlite3 as the database for Active Record
# gem "sqlite3", "~> 1.4" #  <----------- Remove this line

.
.
.

group :production do
  gem 'pg'
end

group :development, :test do
  gem "sqlite3", "~> 1.4"
  # See https://guides.rubyonrails.org/debugging_rails_applications.html#debugging-with-the-debug-gem
  gem "debug", platforms: %i[ mri mswin mswin64 mingw x64_mingw ]
end
```

```bash
bundle install
```

Change the database configuration in `config/database.yml` to use postgresql in the production environment.

```yaml
production:
  adapter: postgresql
  encoding: unicode
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  url: <%= ENV['DATABASE_URL'] %>
```

In your bin directory, create a file called `render-build.sh` and add the following:

```bash
#!/usr/bin/env bash
# exit on error
set -o errexit
bundle install
bundle exec rake db:migrate
```

```bash
#!/usr/bin/env bash
```

This line is called a shebang. It tells the operating system what interpreter to use to run the file. In this case, we are using bash.

```bash
set -o errexit
```

This line tells the operating system to exit the script if any command returns a non-zero status. This is useful because it will prevent the script from continuing if a command fails.

This file will be used to build our application on Render. It will install our gems and run our migrations.

In the root of your project, create a file called `render.yaml`. Render makes the process of deployment easier by simply adding the following:

```yaml
databases:
  - name: betablogclassname
    databaseName: betablogclassname
    user: betablogclassname
    plan: free

services:
  - type: web
    name: betablogclassname
    runtime: ruby
    plan: free
    buildCommand: './bin/render-build.sh'
    # preDeployCommand: "./bin/rails db:migrate" # preDeployCommand only available on paid instance types
    startCommand: './bin/rails server'
    envVars:
      - key: DATABASE_URL
        fromDatabase:
          name: betablogclassname
          property: connectionString
      - key: RAILS_MASTER_KEY
        sync: false
      - key: WEB_CONCURRENCY
        value: 2 # sensible default
```

Change betablogclassname to `beta-blog-classname` or any name you want to use.

Commit to Github.

### Deploying with Render

Keep in mind that you can only have one free service on Render. If you have already deployed a service, you will need to delete it before you can deploy another one.

Sign up for an account at Render. Be sure to use your Github account to sign up to make the process easier.

Once you have signed up, you will be taken to the dashboard.

- Click on the navlink that says "Blueprints"
- Click on the button that says new `blueprint` instance
- Search for the repository you just created and click connect.
- It will ask for a blueprint name. You can name it whatever you want. I will name mine `Default Rails Render`
- Enter the Rails Master Key

The Rails Master Key is a key that is used to decrypt our credentials specific to the environment that we are in. It is a key that is different for every application and environment. It is used to decrypt our credentials.yml.enc file that exists in our config directory.

When we deploy our application, we will need to set the RAILS_MASTER_KEY environment variable to the master key of our application. This will allow our application to decrypt our credentials and use them in the production environment.

Navigate back to your project folder to get the master key. Navigate to `config/master.key` and copy the key.

Once copied, go back to Render and paste the key into the input field.

Click `Apply`. This may take a while.

- Once it is done, go to Dashboard and click on the link to the one that shows the type to be `Web Service`.

Here you will see the URL to your application.

### Changing the Frontend to use the API

In the `environment.ts` file, change the apiUrl to the URL of your application.

```typescript
export const environment = {
  production: true,
  apiUrl: 'https://betablogclassname.onrender.com/',
};
```

Let's change the urls in our authentication service to use the new url.

```typescript
// authentication.service.ts
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { BehaviorSubject } from 'rxjs';
import { Router } from '@angular/router';
import { environment } from '../../../environments/environment';

@Injectable({
	providedIn: 'root',
})
export class AuthenticationService {
	private readonly tokenSubject = new BehaviorSubject<string | null>(null);

	constructor(private http: HttpClient, private router: Router) {}

	login(username: string, password: string) {
		return this.http.post<{ token: string }>(`${environment.apiUrl}/login`, {
			username,
			password,
		});
	}

	signup(data: any) {
		return this.http.post(`${environment.apiUrl}/users`, data);
	}
.
.
.
```

Be sure to push the changes to Github.

### Deploying the Frontend

Use a service like Vercel or Netlify to deploy the frontend.

vercel is a great choice because it is free and it is easy to use. It also has a great integration with Github.

Once you have deployed the application, you can now use it as a live application.

# Any questions?
