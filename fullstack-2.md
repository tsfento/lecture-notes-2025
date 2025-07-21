# Fullstack Project Part 2 - Beta Blogs

## Introduction

- We are going to be incorporating authentication in our Angular frontend
- Handling token-based authentication in our Rails API and frontend Angular App with interceptors
- Builing guards to protect routes in our Angular App

## Live Coding

### Auth Service

Let's start by creating an Auth Service in our Angular App. This service will be responsible for handling the authentication related tasks like login, logout, and checking if the user is authenticated.

```bash
ng g service core/services/authentication
```

```typescript
// authentication.service.ts
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { BehaviorSubject } from 'rxjs';
import { Router } from '@angular/router';

@Injectable({
  providedIn: 'root',
})
export class AuthenticationService {
  private readonly tokenSubject = new BehaviorSubject<string | null>(null);

  constructor(private http: HttpClient, private router: Router) {}

  login(username: string, password: string): Observable<{ token: string }> {
    return this.http.post<{ token: string }>(`${environment.apiUrl}/login`, {
      username,
      password,
    });
  }

  setToken(token: string) {
    localStorage.setItem('token', token);
    this.tokenSubject.next(token);
  }

  getToken() {
    return localStorage.getItem('token');
  }

  isLoggedIn() {
    return !!this.getToken();
  }

  logout() {
    localStorage.removeItem('token');
    this.tokenSubject.next(null);
    this.router.navigate(['/login']);
  }
}
```

### Adding Login

Next, we will start by adding a login form to our Angular App. We will use Angular Reactive Forms to build the login form.

```bash
ng g c features/auth/login
```

```typescript
import { Component } from '@angular/core';
import {
  ReactiveFormsModule,
  FormGroup,
  FormControl,
  Validators,
} from '@angular/forms';
import { Router } from '@angular/router';
import { AuthenticationService } from '../services/authentication.service';

@Component({
  selector: 'app-login',
  standalone: true,
  imports: [ReactiveFormsModule],
  templateUrl: './login.component.html',
  styleUrls: ['./login.component.scss'],
})
export class LoginComponent {
  loginForm: FormGroup = new FormGroup({
    username: new FormControl('', Validators.required),
    password: new FormControl('', Validators.required),
  });

  constructor(
    private authService: AuthenticationService,
    private router: Router
  ) {}

  login() {
    if (this.loginForm.valid) {
      this.authService
        .login(this.loginForm.value.username, this.loginForm.value.password)
        .subscribe({
          next: (res: { token: string }) => {
            console.log('Logged in with token:', res.token);
            this.authService.setToken(res.token);
            this.router.navigate(['/']);
          },
          error: (error: any) => {
            console.error('Login error', error);
          },
        });
    }
  }
}
```

```html
<div class="container">
  <form [formGroup]="loginForm" (ngSubmit)="login()">
    <div class="form-group">
      <label for="username">Username</label>
      <input type="text" formControlName="username" />
    </div>
    <div class="form-group">
      <label for="password">Password</label>
      <input type="password" formControlName="password" />
    </div>
    <button type="submit" [disabled]="!loginForm.valid">Login</button>
  </form>
</div>
```

Add the login route to app.routes.ts

```typescript
import { Routes } from '@angular/router';

export const routes: Routes = [
  {
    path: '',
    pathMatch: 'full',
    loadComponent: () =>
      import('./features/home/home.component').then((c) => c.HomeComponent),
  },
  {
    path: 'login',
    loadComponent: () =>
      import('./features/auth/login/login.component').then(
        (c) => c.LoginComponent
      ),
  },
];
```

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
  margin: 2rem auto;
  padding: 2rem;
  background-color: #f9f9f9;
  border-radius: 8px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  display: flex;
  flex-direction: column;
  gap: 1rem;
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

You can try logging in like so but first create a user.

```bash
User.create(first_name: "jimmy", last_name: "johns", username: "jimmy123", email:"jimmy123@gmail.com", password: "password", password_confirmation: "password").errors.full_messages
```

Then test it out.

### Changing Nav View

Let's inject the authentication service into main-layout to use the logout button.

#### core/components/main-layout

```typsecript
import { Component } from '@angular/core';
import { RouterModule } from '@angular/router';
import { AuthenticationService } from '../../services/authentication.service';

@Component({
  selector: 'app-main-layout',
  standalone: true,
  imports: [RouterModule],
  templateUrl: './main-layout.component.html',
  styleUrls: './main-layout.component.scss'
})
export class MainLayoutComponent {
	constructor(public authService: AuthenticationService) {}

	logout() {
		this.authService.logout();
	}
}
```

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
        >Home</a
      >
      @if (authService.isLoggedIn()){
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

Test out the login and logout.

<!-- GUARDS -->

### Adding Guards

```bash
ng generate guard auth
```

```bash
Which type of guard would you like to create? (Press <space> to select, <a> to toggle all, <i> to invert selection, and <enter> to proceed)
❯◉ CanActivate
 ◯ CanActivateChild
 ◯ CanDeactivate
 ◯ CanMatch
```

Choose CanActivate and press enter.

```typescript
import { inject } from '@angular/core';
import { CanActivateFn, Router } from '@angular/router';
import { AuthenticationService } from './services/authentication.service';

export const authGuard: CanActivateFn = (route, state) => {
  const authService = inject(AuthenticationService);
  const router = inject(Router);

  if (authService.isLoggedIn()) {
    return true;
  } else {
    router.navigate(['/login']);
    return false;
  }
};
```

Add the guard to our routes

#### app.routes.ts

```typescript
import { Routes } from '@angular/router';
import { authGuard } from './auth.guard';

export const routes: Routes = [
  {
    path: '',
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
  },
];
```

### No Auth Guard

```bash
ng generate guard no-auth
```

```bash
germancruz@Code-Coach-Three fe_todo_list_v2 % ng generate guard no-auth
? Which type of guard would you like to create? (Press <space> to select, <a> to toggle all, <i> to invert selection, and <enter> to proceed)
❯◉ CanActivate
 ◯ CanActivateChild
 ◯ CanDeactivate
 ◯ CanMatch
```

Choose CanActivate and press enter.

```typescript
import { inject } from '@angular/core';
import { CanActivateFn, Router } from '@angular/router';
import { AuthenticationService } from './services/authentication.service';

export const noAuthGuard: CanActivateFn = (route, state) => {
  const authService = inject(AuthenticationService);
  const router = inject(Router);

  if (authService.isLoggedIn()) {
    router.navigate(['/']);
    return false;
  } else {
    return true;
  }
};
```

Apply this guard to the login route.

#### app.routes.ts

```typescript
import { Routes } from '@angular/router';
import { authGuard } from './auth.guard';
import { noAuthGuard } from './no-auth.guard';

export const routes: Routes = [
  {
    path: '',
    loadComponent: () => import('./features/home/home.component').then((c) => c.HomeComponent),
    canActivate: [authGuard],
  },
  {
    path: 'login',
    loadComponent: () => import('./features/auth/login/login.component').then((c) => c.LoginComponent),
    canActivate: [noAuthGuard],
  },
];
```

Login and test the guards.

# Any questions?
