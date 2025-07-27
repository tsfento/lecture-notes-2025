# Fullstack Project Part 4 - Beta Blogs

## Introduction

- We are going to be adding signup to our frontend

## Live Coding

### Sign Up

```bash
ng g c features/auth/signup
```

Add a route to app.routes.ts for the component for signup.

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
    path: 'signup',
    loadComponent: () =>
      import('./features/auth/signup/signup.component').then(
        (c) => c.SignupComponent
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

In the main layout component, let's add the signup route to the navigation.

#### core/components/main-layout.component.html

```html
<nav>
  <img src="/assets/images/logo.png" alt="logo" width="100px" />
  <a
    routerLink="/"
    routerLinkActive="active"
    [routerLinkActiveOptions]="{exact: true}"
    >Home</a
  >
  @if (authService.isLoggedIn()){
  <a routerLink="/blogs/new" routerLinkActive="active">Create Blog</a>
  <a (click)="logout()">Logout</a>
  }@else {
  <a routerLink="/login">Login</a>
  <a routerLink="/signup">Sign Up</a>
  }
</nav>
```

Create a shared.styles.scss file in the features/auth directory and copy the scss from the login component into it.

#### features/auth/shared.styles.scss

```scss
/* login.component.scss */
.container {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: calc(
    100vh - 40px
  ); /* Full viewport height to ensure vertical centering */
  background-color: #f0f0f0; /* Light background for the overall page */
}

form {
  max-width: 400px;
  width: 400px;
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
input[type='password'] {
  padding: 0.75rem;
  border: 1px solid #ccc;
  border-radius: 4px;
  font-size: 1rem;
  color: #222; /* Dark gray for the input text */
  background-color: white;
  transition: border-color 0.3s;
}

input[type='text']:focus,
input[type='password']:focus {
  border-color: #007bff; /* Light blue border for focus */
  outline: none;
}

button[type='submit'] {
  padding: 0.75rem;
  border: none;
  border-radius: 4px;
  background-color: #eeb543; /* Yellow background for the submit button */
  color: white;
  font-size: 1rem;
  font-weight: bold;
  cursor: pointer;
  transition: background-color 0.3s;
}

button[type='submit']:hover {
  background-color: #d4a03f; /* A darker shade of yellow on hover */
}
```

#### features/auth/signup/signup.component.ts

```typescript
import { Component } from '@angular/core';
import {
  FormControl,
  FormGroup,
  ReactiveFormsModule,
  Validators,
} from '@angular/forms';

@Component({
  selector: 'app-signup',
  standalone: true,
  imports: [ReactiveFormsModule],
  templateUrl: './signup.component.html',
  styleUrl: '../shared.styles.scss',
})
export class SignupComponent {
  signupForm = new FormGroup({
    email: new FormControl('', Validators.required),
    first_name: new FormControl('', Validators.required),
    last_name: new FormControl('', Validators.required),
    username: new FormControl('', Validators.required),
    password: new FormControl('', Validators.required),
    password_confirmation: new FormControl('', Validators.required),
  });
}
```

#### features/auth/signup/signup.component.html

```html
<div class="container">
  <form [formGroup]="signupForm" (ngSubmit)="signup()">
    <div class="form-group">
      <label for="email">Email</label>
      <input type="text" formControlName="email" />
    </div>
    <div class="form-group">
      <label for="first_name">First Name</label>
      <input type="text" formControlName="first_name" />
    </div>
    <div class="form-group">
      <label for="last_name">Last Name</label>
      <input type="text" formControlName="last_name" />
    </div>
    <div class="form-group">
      <label for="username">Username</label>
      <input type="text" formControlName="username" />
    </div>
    <div class="form-group">
      <label for="password">Password</label>
      <input type="password" formControlName="password" />
    </div>
    <div class="form-group">
      <label for="password_confirmation">Confirm Password</label>
      <input type="password" formControlName="password_confirmation" />
    </div>
    <button type="submit" [disabled]="!signupForm.valid">Sign Up</button>
  </form>
</div>
```

Let's add the method to the auth service to sign up a user.

#### core/services/authentication.service.ts

```typescript
signup(data: any) {
  return this.http.post(`${environment.apiUrl}/users`, data);
}
```

From there let's add the signup method to the signup component.

```typescript
import { Component } from '@angular/core';
import {
  FormControl,
  FormGroup,
  ReactiveFormsModule,
  Validators,
} from '@angular/forms';
import { AuthenticationService } from '../../../core/services/authentication.service';
import { Router } from '@angular/router';

@Component({
  selector: 'app-signup',
  standalone: true,
  imports: [ReactiveFormsModule],
  templateUrl: './signup.component.html',
  styleUrl: '../shared.styles.scss',
})
export class SignupComponent {
  signupForm = new FormGroup({
    email: new FormControl('', Validators.required),
    first_name: new FormControl('', Validators.required),
    last_name: new FormControl('', Validators.required),
    username: new FormControl('', Validators.required),
    password: new FormControl('', Validators.required),
    password_confirmation: new FormControl('', Validators.required),
  });

  constructor(
    private authService: AuthenticationService,
    private router: Router
  ) {}

  signup() {
    this.authService.signup(this.signupForm.value).subscribe({
      next: (data) => {
        console.log(data);
        this.router.navigate(['/login']);
      },
      error: (error) => {
        console.error(error);
      },
    });
  }
}
```

# Any questions?
