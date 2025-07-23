# Fullstack Project Part 3 - Beta Blogs

## Introduction

- We are going to be adding the ability to create a blog from the frontend
- We are going to use interceptors to pass our token to the backend

## Live Coding

### Creating a Blog

Let's add a route so logged in users can create a blog.

```html
<div class="layout-container">
  <div class="sidebar">
    <!-- Sidebar content -->
    <nav>
      <img src="/assets/images/logo.png" alt="logo" width="100px" />
      <a
        routerLink="/"
        routerLinkActive="active-link"
        [routerLinkActiveOptions]="{exact: true}"
        >Home
      </a>
      @if (authService.isLoggedIn()){
      <a
        routerLink="/blogs/new"
        routerLinkActive="active-link"
        [routerLinkActiveOptions]="{ exact: true }"
        >Create Blog</a
      >
      <a (click)="logout()">Logout</a>
      }@else {
      <a routerLink="/login">Login</a>
      }
    </nav>
  </div>
  <div class="content">
    <router-outlet></router-outlet>
  </div>
</div>
```

Let's create a component for a form that will allow us to create a blog.

```bash
ng g c features/blogs/blogs-new
```

Let's add a route to app.routes.ts for the component

```typescript
import { Routes } from '@angular/router';

export const routes: Routes = [
  {
    path: '',
    pathMatch: 'full',
    loadComponent: () =>
      import('./features/home/home.component').then((c) => c.HomeComponent),
    canActivate: [authGuard],
  },
  {
    path: 'login',
    loadComponent: () =>
      import('./features/auth/login/login.component').then(
        (c) => c.LoginComponent
      ),
    canActivate: [noAuthGuard],
  },
  {
    path: 'blogs/new',
    loadComponent: () =>
      import('./features/blogs/blogs-new/blogs-new.component').then(
        (c) => c.BlogsNewComponent
      ),
    canActivate: [authGuard],
  },
];
```

Let's go to our blog service and add a method to create a blog.

```typescript
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

  // create blog
 createBlog(blog: { title: string; content: string }): Observable<Blog> {
  return this.http.post<Blog>(`${environment.apiUrl}/blogs`, blog);
}
```

Let's add the method to the component.

```typescript
import { Component } from '@angular/core';
import {
  FormControl,
  FormGroup,
  ReactiveFormsModule,
  Validators,
} from '@angular/forms';
import { BlogService } from '../../../core/services/blog.service';
import { Blog } from '../../../shared/models/blog';
import { Router } from '@angular/router';

@Component({
  selector: 'app-blogs-new',
  standalone: true,
  imports: [ReactiveFormsModule],
  templateUrl: './blogs-new.component.html',
  styleUrls: './blogs-new.component.scss',
})
export class BlogsNewComponent {
  blogForm = new FormGroup({
    title: new FormControl('', [Validators.required]),
    content: new FormControl('', [Validators.required]),
  });

  constructor(private blogService: BlogService, private router: Router) {}

  onSubmit() {
    // call create blog service
    this.blogService.createBlog(this.blogForm.value).subscribe({
      next: (blog: Blog) => {
        console.log('Blog created', blog);
        this.router.navigate(['/']);
      },
      error: (error: any) => {
        console.error('Error creating blog', error);
      },
    });
  }
}
```

Let's move on to the UI for the create blog component.

```html
<!-- blogs-new.component.html -->
<div class="container">
  <h1 class="create-blog-title">Create a Blog!</h1>
  <form [formGroup]="blogForm" (ngSubmit)="onSubmit()" class="form">
    <div class="form-group">
      <label for="title">Title:</label>
      <input type="text" formControlName="title" id="title" required />
    </div>
    <div class="form-group">
      <label for="content">Content:</label>
      <textarea
        formControlName="content"
        id="content"
        rows="5"
        required
      ></textarea>
    </div>
    <button type="submit" [disabled]="!blogForm.valid">Submit</button>
  </form>
</div>
```

```scss
// blogs-new.component.scss
.create-blog-title {
  color: #545454;
  text-align: center;
}

form {
  max-width: 400px;
  margin: 2rem auto;
  padding: 2rem;
  background-color: #f9f9f9;
  border-radius: 8px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.form-group {
  display: flex;
  flex-direction: column;
}

label {
  margin-right: 0.5rem;
}

input[type='text'],
textarea {
  padding: 0.75rem;
  border: 1px solid #ccc;
  border-radius: 4px;
  font-size: 1rem;
  color: #222;
  background-color: white;
  transition: border-color 0.3s;
}

input[type='text']:focus,
textarea:focus {
  border-color: #007bff;
  outline: none;
}

button[type='submit'] {
  padding: 0.75rem;
  border: none;
  border-radius: 4px;
  background-color: #eeb543;
  color: white;
  font-size: 1rem;
  font-weight: bold;
  cursor: pointer;
  transition: background-color 0.3s;
}

button[type='submit']:hover {
  background-color: #d4a03f;
}
```

When we create a blog, we get a 401 error. Let's add an interceptor to handle the token.

### Token Interceptor

Let's create an interceptor to handle the token.

```bash
ng generate interceptor auth-token
```

```typescript
import { HttpInterceptorFn } from '@angular/common/http';
import { inject } from '@angular/core';
import { AuthenticationService } from './core/services/authentication.service';

export const authTokenInterceptor: HttpInterceptorFn = (req, next) => {
  const authService = inject(AuthenticationService);
  const authToken = authService.getToken();

  const authReq = authToken
    ? req.clone({
        headers: req.headers.set('Authorization', `Bearer ${authToken}`),
      })
    : req;
  return next(authReq);
};
```

Add the interceptor in the app.config.ts file.

```typescript
import { ApplicationConfig } from '@angular/core';
import { provideRouter } from '@angular/router';

import { routes } from './app.routes';
import { provideHttpClient, withInterceptors } from '@angular/common/http';
import { authTokenInterceptor } from './auth-token.interceptor';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
    provideHttpClient(withInterceptors([authTokenInterceptor])),
  ],
};
```

When we test it out, we may very well get an error. That's because previously when creating a blog, we didn't account for token-based authentication. Which we can use to create a blog.

Navigate to the backend and add the following to the blog controller.

```ruby
class BlogsController < ApplicationController
  before_action :authenticate_request, except: [:index]

  def index
    blogs = Blog.all

    render json: BlogBlueprint.render(blogs, view: :normal)
  end

  def create
    blog = @current_user.blogs.new(blog_params)

    if blog.save
      render json: BlogBlueprint.render(blog, view: :normal), status: :created
    else
      render json: blog.errors, status: :unprocessable_entity
    end
  end

  private

  def blog_params
    params.permit(:title, :content)
  end
end
```

Here we removed the user id from blog params and made sure the current user is creating the blog.

Great, now we can create a blog.

# Any questions?
