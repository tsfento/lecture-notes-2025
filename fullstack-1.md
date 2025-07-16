# Fullstack Project Part 1 - Beta Blogs

## Introduction

-   We are going to be making an Angular frontend to link to our Rails api

## Live Coding

This project uses Angular 19 for the frontend and Rails 8 for the backend.

We will be using Angular 19 for the frontend of our project.

```bash
germancruz@codecoachthree ruby-on-rails-projects-codelabs % ng new fe_beta_blogs
? Which stylesheet format would you like to use? SCSS   [ https://sass-lang.com/documentation/syntax#scss                ]

? Do you want to enable Server-Side Rendering (SSR) and Static Site Generation (SSG/Prerendering)? No
```

Add the following global styles to styles.scss:

```css
*,
html,
body {
  margin: 0;
  padding: 0;
}
```

### Setting up Models

```bash
ng g class shared/models/user
```

```javascript
export class User {
	id!: number;
	username: string = '';
	email: string = '';
	first_name: string = '';
	last_name: string = '';
}
```

```bash
ng g class shared/models/blog
```

```javascript
import { User } from './user';

export class Blog {
	id!: number;
	title: string = '';
	content: string = '';
	user_id?: number;
	user?: User;
}
```

```bash
ng g c features/home --standalone
```

```bash
ng g c shared/blogs/blog-list --standalone
```

```bash
ng g s core/services/blog
```

```bash
ng g environments
```

### Setting up Environments

#### environments/environment.prod.ts

```javascript
export const environment = {
	production: true,
	apiUrl: 'https://www.production.com',
};
```

#### environments/environment.ts

```javascript
export const environment = {
	production: false,
	apiUrl: 'http://localhost:3000',
};
```

#### core/services/blog.service.ts

```javascript
import { Injectable } from '@angular/core';
import { Observable } from 'rxjs';
import { Blog } from '../../shared/models/blog';
import { HttpClient } from '@angular/common/http';
import { environment } from '../../../environments/environment';

@Injectable({
	providedIn: 'root',
})
export class BlogService {
	constructor(private http: HttpClient) {}

	getBlogs(): Observable<Blog[]> {
		return this.http.get<Blog[]>(`${environment.apiUrl}/blogs`);
	}
}
```

Since we are using HttpClient, we need to configure our app.config.ts file.

#### app.config.ts

```javascript
import { ApplicationConfig } from '@angular/core';
import { provideRouter } from '@angular/router';

import { routes } from './app.routes';
import { provideHttpClient } from '@angular/common/http';

export const appConfig: ApplicationConfig = {
	providers: [provideRouter(routes), provideHttpClient()],
};
```

Let’s add routes to our app:

#### app.routes.ts

```javascript
import { Routes } from '@angular/router';

export const routes: Routes = [
	{
		path: '',
		pathMatch: 'full',
		loadComponent: () => import('./features/home/home.component').then((c) => c.HomeComponent),
	},
];
```

#### app.component.ts

```html
<router-outlet />
```

#### features/home/home.component

```javascript
import { Component, OnInit } from '@angular/core';
import { BlogService } from '../../core/services/blog.service';

@Component({
	selector: 'app-home',
	standalone: true,
	imports: [],
	templateUrl: './home.component.html',
	styleUrl: './home.component.scss',
})
export class HomeComponent implements OnInit {
	constructor(private blogsService: BlogService) {}

	ngOnInit(): void {
		this.blogsService.getBlogs().subscribe({
			next: (blogs) => {
				console.log(blogs);
			},
			error: (error) => {
				console.error(error);
			},
		});
	}
}
```

### CORS issues

```bash
localhost/:1 Access to XMLHttpRequest at 'http://localhost:3000/blogs' from origin 'http://localhost:4200' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource.
home.component.ts:21 HttpErrorResponse {headers: _HttpHeaders, status: 0, statusText: 'Unknown Error', url: 'http://localhost:3000/blogs', ok: false, …}
error @ home.component.ts:21
```
This is because the Rails API is not allowing requests from the Angular front end. This is due to there being what is called a CORS issue. CORS is a security feature that prevents requests from other domains from accessing the API's resources.

We need to configure our Rails API to allow requests from our Angular app.

Stop the server and add the following to your Gemfile.

```ruby
gem "rack-cors"
```

```bash
bundle install
```

#### config/initializers/cors.rb

```ruby
Rails.application.config.middleware.insert_before 0, Rack::Cors do
  allow do
    origins 'http://localhost:4200'
    resource '*', headers: :any, methods: %i[get post put patch delete options head]
  end
end
```

Start the server again.

Make sure the index action of the BlogsController does not require authentication.

```ruby
class BlogsController < ApplicationController
  before_action :authenticate_request, except: [:index]
```

### Displaying blogs

In the home component, we will display the blogs.

```javascript
import { Component, OnInit } from '@angular/core';
import { BlogService } from '../../core/services/blog.service';
import { BlogListComponent } from '../../shared/blogs/blog-list/blog-list.component';

@Component({
  selector: 'app-home',
  standalone: true,
  imports: [BlogListComponent], // Add this line
  templateUrl: './home.component.html',
  styleUrl: './home.component.scss'
})
```

#### home.component.html

```html
<app-blog-list />

<!-- Notice that nothing displays yet. That's because we haven't passed any data to our component. -->
```

#### shared/blogs/blog-list/blog-list.component.ts

```javascript
import { Component, Input } from '@angular/core';
import { Blog } from '../../models/blog';

@Component({
	selector: 'app-blog-list',
	standalone: true,
	imports: [],
	templateUrl: './blog-list.component.html',
	styleUrl: './blog-list.component.scss',
})
export class BlogListComponent {
	@Input({ required: true }) blogs: Blog[] = [];
}
```

In the home component, let's pass the blogs to the blog list component.

#### features/home/home.component.ts

```javascript
import { Component, OnInit } from '@angular/core';
import { BlogService } from '../../core/services/blog.service';
import { BlogListComponent } from '../../shared/blogs/blog-list/blog-list.component';
import { Blog } from '../../shared/models/blog';

@Component({
	selector: 'app-home',
	standalone: true,
	imports: [BlogListComponent],
	templateUrl: './home.component.html',
	styleUrl: './home.component.scss',
})
export class HomeComponent implements OnInit {
	homeBlogs: Blog[] = [];

	constructor(private blogsService: BlogService) {}

	ngOnInit(): void {
		this.blogsService.getBlogs().subscribe({
			next: (blogs) => {
				this.homeBlogs = blogs;
			},
			error: (error) => {
				console.error(error);
			},
		});
	}
}
```

#### features/home/home.component.html

```html
<app-blog-list [blogs]="homeBlogs" />
```

Let’s adjust the HTML to display the blogs:

#### shared/blogs/blog-list/blog-list.component.html

```html
@defer(on immediate) {
<div class="blog-container">
  @for (blog of blogs; track blog.id) {
  <div
    class="blog-preview-container"
  >
    <div class="blog-preview-content">
      <h2>{{ blog.title }}</h2>
      <p>{{ blog.user?.username }}</p>
      <p>{{ blog.content }}</p>
    </div>
    <img
      src="https://assets.bitdegree.org/online-learning-platforms/storage/media/2018/08/how-to-become-a-programmer.jpg"
      alt="blog image"
      class="blog-image"
    />
  </div>
  }
</div>
}
```

Style the blog-container

#### shared/blogs/blog-list/blog-list.component.scss

```css
/* ParentComponent.scss */
.blog-container {
  display: flex;
  flex-wrap: wrap;
  gap: 20px; /* Adjust gap between items as needed */
  justify-content: center; /* Center the items horizontally */
  align-items: flex-start; /* Align items to the start of the flex container */
}

.blog-preview-container{
  display: flex;
  justify-content: space-between;
  width: 100%;
  background-color: #f5f5f5; /* Light grey background for subtlety */
  border-radius: 1rem; /* Rounded corners for a modern look */
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1); /* Slightly deeper shadow for emphasis */
}

.blog-preview-content {
  flex-grow: 1;
  padding: 20px; /* Add padding for spacing */
}

.blog-preview-content h2 {
  font-family: Roboto, sans-serif;
  font-size: 1.5rem;
  margin: 0.5rem 0;
}

.blog-preview-content p {
  font-family: Roboto, sans-serif;
  color: #333;
  line-height: 1.4;
}

.blog-image {
  width: 100%;
  max-width: 800px;
  object-fit: cover;
  border-top-right-radius: 1rem;
  border-bottom-right-radius: 1rem;
}
```

### App Layout and Styling

```bash
ng g c core/components/main-layout --standalone
```

```javascript
import { Component } from '@angular/core';
import { RouterModule } from '@angular/router';

@Component({
  selector: 'app-main-layout',
  standalone: true,
  imports: [RouterModule],
  templateUrl: './main-layout.component.html',
  styleUrl: './main-layout.component.scss'
})
export class MainLayoutComponent {

}
```

```html
<div class="layout-container">
  <div class="sidebar">
    <!-- Sidebar content -->
    <nav>
      <a routerLink="/" routerLinkActive="active-link" [routerLinkActiveOptions]="{exact: true}">Home</a>
    </nav>
  </div>
  <div class="content">
    <router-outlet></router-outlet>
  </div>
</div>
```

```css
.layout-container {
  display: flex;
  min-height: 100vh; /* Full viewport height */
}

.sidebar {
  position: fixed; /* Fix sidebar to the viewport */
  top: 0; /* Align sidebar to the top of the viewport */
  left: 0; /* Align sidebar to the left of the viewport */
  width: 210px; /* Sidebar width */
  height: 100vh; /* Full viewport height */
  overflow-y: auto; /* Enable scrolling if content overflows */
  background-color: #222; /* Light grey background */
  padding: 20px;
}

.content {
  margin-left: 250px; /* Offset content to the right of the sidebar */
  flex-grow: 1; /* Take up the remaining space */
  overflow-y: auto; /* Scrollable content area */
  padding: 20px;
  height: calc(100vh - 40px); /* Optional: Set to full viewport height */
  overflow-y: scroll; /* Optional: Always enable vertical scrolling */
}
```

- ```overflow-y: auto```: Adds a vertical scrollbar only when the content overflows the element's height.

- ```overflow-y: scroll```: Always shows a vertical scrollbar, even when the content doesn't overflow. On some browsers, this can reserve space for the scrollbar, avoiding layout shifts when content grows.

```css
.sidebar nav {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.sidebar nav a {
  text-decoration: none; /* Remove underline from links */
  font-family: sans-serif;
  color: white; /* Black text color for links */
  padding: 10px 15px; /* Padding for larger click area and spacing */
  margin: 5px 0; /* Margin between links */
  border-radius: 4px; /* Rounded corners for a modern look */
  transition: background-color 0.3s ease; /* Smooth transition for hover effect */
  cursor:pointer;
}

.sidebar nav a:hover {
  background-color: #ffcc00; /* Yellow background on hover */
  color: #000; /* Optional: Change text color on hover if needed */
}

/* Active link styling */
.sidebar nav a.active {
  background-color: #ffcc00; /* Yellow background for active link */
  color: #fff; /* White text for contrast */
}
```

#### app.component.ts

```javascript
import { Component } from '@angular/core';
import { RouterOutlet } from '@angular/router';
import { MainLayoutComponent } from './core/components/main-layout/main-layout.component';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [ MainLayoutComponent],
  templateUrl: './app.component.html',
  styleUrl: './app.component.scss'
})
export class AppComponent {
  title = 'fe_beta_blogs';
}
```

#### app.component.html

```html
<app-main-layout/>
```

# Any questions?