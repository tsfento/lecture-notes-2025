# Real Time Communication with Pusher

## Introduction

- We will be adding real-time communication to our app using Pusher

## Live Coding

In this live coding session, we will be building a real-time communication app using Ruby on Rails and Pusher. We will be using the Pusher gem to integrate Pusher into our Rails application.

Let's go ahead and add the Pusher gem to our Rails application. We will use the pusher gem, which provides a Ruby client for the Pusher Channels service.

Add the pusher gem to your `Gemfile` to include it in all environments

```ruby
gem 'pusher'
```

Then run `bundle install` to install the gem.

Navigate to `App Keys` and you will be able to obtain the following credentials:

- `app_id`
- `key`
- `secret`
- `cluster`

We'll use the primary encryption file to store our Pusher credentials. Which means both the development and production environment will use the same credentials.

```bash
EDITOR="code --wait" bin/rails credentials:edit
```

Add the Pusher credentials to the credentials file. For this demo, we’ll use the main `credentials.yml.enc` file, but for real projects, it’s best practice to use separate credentials for each environment (development, test, and production) to keep your secrets secure and environments isolated.

```yaml
pusher:
  app_id: '17536f051'
  key: '6ed6as5d0fc1fff140be76a'
  secret: '03d4a2c9c01f31f8e4064d'
  cluster: 'us2'
```

Let's now configure Pusher in our Rails API. Create a new file called `pusher.rb` in the `config/initializers` directory:

```ruby
require 'pusher'

Pusher.app_id = Rails.application.credentials.pusher[:app_id]
Pusher.key = Rails.application.credentials.pusher[:key]
Pusher.secret = Rails.application.credentials.pusher[:secret]
Pusher.cluster = Rails.application.credentials.pusher[:cluster]
Pusher.logger = Rails.logger
```

Great we have configured Pusher in our Rails API.

Let's start using Pusher

### Using Pusher to Broadcast Messages

We will be using Pusher to broadcast when a user likes a new blog. The creator of the blog will be notified when a user likes their blog.

```bash
rails g model Like user:references likeable:references{polymorphic}
```

```bash
rails db:migrate
```

Change User to have many likes

```ruby
class User < ApplicationRecord
  has_many :blogs
  has_one :profile
  has_many :likes

  after_create :create_profile

  validates :username, :email, :first_name, :last_name, presence: true
  has_secure_password
end
```

The `Like` model will include this for us already:

```ruby
class Like < ApplicationRecord
  belongs_to :user
  belongs_to :likeable, polymorphic: true
end
```

We will add the following to the `Blog` model.

Instead of using `likeable` we will use likes to make it easier to understand.

```ruby
class Blog < ApplicationRecord
  include Rails.application.routes.url_helpers

  has_and_belongs_to_many :categories
  belongs_to :user
  has_many :likes, as: :likeable

  validates :title, :content, presence: true

  has_one_attached :cover_image

  def cover_image_url
    rails_blob_url(self.cover_image, only_path: false) if self.cover_image.attached?
  end
end
```

Let's also include the routes for liking a blog or unliking a blog;

```ruby
Rails.application.routes.draw do
  post '/login', to: 'sessions#create'
  resources :blogs do
    post 'like'
    delete 'unlike'
  end
  resources :users, only: [:create]
end
```

We will add the following to the `BlogsController`:

```ruby
def like
  blog = Blog.find(params[:blog_id])
  like = blog.likes.new(user_id: @current_user.id)
  blog_creator = blog.user

  if like.save
    Pusher.trigger(blog_creator.id, 'like', {
      blog_id: blog.id,
      notification: "#{@current_user.username} has liked #{blog.title}!"
    })
    head :ok
  else
    render json: nil, status: :unprocessable_entity
  end
end

def unlike
  blog = Blog.find(params[:blog_id])
  like = blog.likes.find_by(user_id: @current_user.id)
  if like.destroy
    head :ok
  else
    render json: nil, status: :unprocessable_entity
  end
end
```

Be sure to make sure the user is authenticated before requesting the blogs.

```ruby
class BlogsController < ApplicationController
  before_action :authenticate_request

  def index
```

Since our index action is serializing the blogs with the view blueprint..

```ruby
def index
  blogs = Blog.all

  render json: BlogBlueprint.render(blogs, view: :normal)
end
```

Let's change the normal view to include the count for likes for the blog blue print.

```ruby
view :normal do
  field :likes_count do |blog|
    blog.likes.count
  end
  association :user, blueprint: UserBlueprint, view: :normal
end
```

### Checking if a user liked the blog

We will add a method to the Blog model to check if a user liked a blog.

```ruby
def liked?(user)
  self.likes.where(user: user).exists?
end
```

In our blog blueprint let's change the following to include the liked field.

```ruby
view :normal do
  field :liked do |blog, options|
    blog.liked?(options[:current_user])
  end
  field :likes_count do |blog|
    blog.likes.count
  end
  association :user, blueprint: UserBlueprint, view: :normal
end
```

Now in the `BlogsController` we will include the current user in the options.

```ruby
def index
  blogs = Blog.all

  render json: BlogBlueprint.render(blogs, view: :normal, current_user: @current_user)
end
```

### Front End Implementation

We will add icons for like and unlike.

There are various ways to implement this, we will use angular material for this

```bash
germancruz@codecoachthree fe_beta_blogs % ng add @angular/material

ℹ Using package manager: npm
✔ Found compatible package version: @angular/material@17.2.2.
✔ Package information loaded.

The package @angular/material@17.2.2 will be installed and executed.
Would you like to proceed? Yes
✔ Packages successfully installed.
? Choose a prebuilt theme name, or "custom" for a custom theme: Indigo/Pink        [ Preview: https://material.angular.io?theme=indigo-pink ]
? Set up global Angular Material typography styles? No
? Include the Angular animations module? Do not include
```

Let's go ahead and navigate to `blog-list.component.html`.

```html
<div class="blog-preview-container">
  <div class="blog-preview-content">
    @if (blog.liked) {
    <mat-icon>favorite</mat-icon>
    }@else {
    <mat-icon>favorite_border</mat-icon>
    }
  </div>
</div>
```

Let's include the MatIconModule in the `blog-list.component.ts` file

```typescript
import { Component, Input } from '@angular/core';
import { Blog } from '../../models/blog';
import { MatIconModule } from '@angular/material/icon';
@Component({
  selector: 'app-blog-list',
  standalone: true,
  imports: [MatIconModule],
  templateUrl: './blog-list.component.html',
  styleUrl: './blog-list.component.scss',
})
```

Be sure to include likes_count and liked in the blog model

```typescript
import { User } from './user';

export class Blog {
  id!: number;
  title: string = '';
  content: string = '';
  cover_image_url: string = '';
  liked?: boolean = false;
  likes_count?: number = 0;
  user_id?: number;
  user?: User;
}
```

Let's go back to our template file and include click listeners to trigger requests

```html
@if (blog.liked) {
<mat-icon (click)="toggleLiked(blog.id)">favorite</mat-icon>
}@else {
<mat-icon (click)="toggleLiked(blog.id)">favorite_border</mat-icon>
}
```

#### blog-list.component.ts

Let's fix the toggle method to update the like both on the front end and back end.

```typescript
@Output() onChangeBlogs: EventEmitter<Blog[]> = new EventEmitter<Blog[]>();
.
.
.
  toggleLiked(blogId: number) {
    const blogs = [...this.blogs];
    const blogIndex = blogs.findIndex((blog) => blog.id === blogId);

    if (blogIndex === -1) return;

    const blog = blogs[blogIndex];
    const isLiked = blog.liked;
    const blog$ = isLiked
      ? this.blogService.unLikeBlog(blogId)
      : this.blogService.likeBlog(blogId);

    blog.liked = !isLiked;

    blog$.subscribe({
      next: () => {
        blog.likes_count! += isLiked ? -1 : 1;
        this.onChangeBlogs.emit(blogs);
      },
      error: (error) => {
        console.error('Error toggling like status:', error);
        blog.liked = isLiked;
      },
    });
  }
```

#### home.component.html

```html
<app-blog-list [blogs]="homeBlogs" (onChangeBlogs)="homeBlogs = $event" />
```

Include the following methods in the blog service.

```typescript
likeBlog(blogId: number): Observable<Blog> {
  return this.http.post<Blog>(`${environment.apiUrl}/blogs/${blogId}/like`, {});
}

unLikeBlog(blogId: number): Observable<Blog> {
  return this.http.delete<Blog>(`${environment.apiUrl}/blogs/${blogId}/unlike`, {});
}
```

Test the like and unlike and make sure the console logs the correct response.

### Real Time Communication with Pusher on the Front End

Let's go ahead and include the Pusher library in our Angular application.

```bash
npm install pusher-js
```

Go ahead and include the key and cluster in the environment file.

#### environment.development.ts

```typescript
export const environment = {
  production: false,
  apiUrl: 'http://localhost:3000',
  pusher: {
    key: '2b43c034d9a88',
    cluster: 'us2',
  },
};
```

#### environment.ts

```typescript
export const environment = {
  production: true,
  apiUrl: 'https://betablogclassname.onrender.com/',
  pusher: {
    key: '2b43c054887f229d9a88',
    cluster: 'us2',
  },
};
```

Let's create a notification service to handle the pusher events.

```bash
ng g s core/services/notification
```

```typescript
import { Injectable } from '@angular/core';
import { Subject } from 'rxjs';
import Pusher from 'pusher-js';
import { environment } from '../../../environments/environment';

@Injectable({
  providedIn: 'root',
})
export class NotificationService {
  // Create a subject to listen to the pusher event
  notification$: Subject<any> = new Subject<any>();
  pusher: any;
  channel: any;

  constructor() {}

  // Listen to the pusher event
  listen(userId: number) {
    this.pusher = new Pusher(environment.pusher.key, {
      cluster: environment.pusher.cluster,
    });
    // Subscribe to the channel, using the user id
    this.channel = this.pusher.subscribe(userId.toString());

    // Listen to the like event made by the API
    this.channel.bind('like', (data: any) => {
      // Set the notification to the data received from the event
      this.setNotification(data.notification);
    });
  }

  setNotification(notifications: any): void {
    this.notification$.next(notifications);
  }
}
```

### Bootstrap the User

In order for us to listen to subscribe to the specific user channel, we will need to include the user id. The problem is, we don't have the user id at the start of the application.

Let's create a service to request this information from the API.

```bash
ng g s core/services/user
```

```typescript
import { Injectable } from '@angular/core';
import { BehaviorSubject, tap } from 'rxjs';
import { User } from '../../shared/models/user';
import { HttpClient } from '@angular/common/http';
import { environment } from '../../../environments/environment';

@Injectable({
  providedIn: 'root',
})
export class UserService {
  currentUserSubject = new BehaviorSubject<User | null>(null);

  constructor(private http: HttpClient) {}

  setCurrentUser(user: User | null) {
    this.currentUserSubject.next(user);
  }

  getBootstrapData() {
    return this.http.get(`${environment.apiUrl}/web/bootstrap`).pipe(
      tap((data: any) => {
        this.setCurrentUser(data.current_user);
      })
    );
  }
}
```

In our API lets create a web controller to handle the bootstrap data.

#### routes.rb

```ruby
scope '/web' do
  get 'bootstrap', to: 'web#bootstrap'
end
```

Then create a `app/controllers/web_controller`

```ruby
class WebController < ApplicationController
  before_action :authenticate_request

  def bootstrap
    render json: UserBlueprint.render(@current_user, view: :normal), status: :ok
  end
end
```

Now let's initialize the user during app initialization in our `app.config.ts` file.

```typescript
import { APP_INITIALIZER, ApplicationConfig } from '@angular/core';
import { provideRouter } from '@angular/router';

import { routes } from './app.routes';
import { provideHttpClient, withInterceptors } from '@angular/common/http';
import { authTokenInterceptor } from './auth-token.interceptor';
import { UserService } from './core/services/user.service';
import { AuthenticationService } from './core/services/authentication.service';
import { of } from 'rxjs';
import { NotificationService } from './core/services/notification.service';
import { User } from './shared/models/user';
import { provideAnimations } from '@angular/platform-browser/animations';

export function initializeUserData(
  userService: UserService,
  authService: AuthenticationService,
  notificationService: NotificationService
) {
  if (authService.isLoggedIn()) {
    return () =>
      userService.getBootstrapData().subscribe({
        next: (user: User) => {
          console.log(user);
          notificationService.listen(user.id);
        },
        error: () => authService.logout(),
      });
  } else {
    return () => of(null);
  }
}

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
    provideHttpClient(withInterceptors([authTokenInterceptor])),
    {
      provide: APP_INITIALIZER,
      useFactory: initializeUserData,
      deps: [UserService, AuthenticationService, NotificationService],
      multi: true,
    },
  ],
};
```

Now that we are listening to the pusher event, we can go ahead and include the notification in the app component to allow the user to see notifications at all times.

We will use the MatSnackBarModule to display the notifications.

#### app.component.ts

```typescript
import { Component, OnInit } from '@angular/core';
import { MainLayoutComponent } from './core/components/main-layout/main-layout.component';
import { NotificationService } from './core/services/notification.service';
import { MatSnackBar, MatSnackBarModule } from '@angular/material/snack-bar';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [MainLayoutComponent, MatSnackBarModule],
  templateUrl: './app.component.html',
  styleUrl: './app.component.scss',
})
export class AppComponent implements OnInit {
  constructor(
    private notificationService: NotificationService,
    private snackBar: MatSnackBar
  ) {}

  ngOnInit(): void {
    this.notificationService.notification$.subscribe((notification) => {
      this.snackBar.open(notification, 'Dismiss', {
        duration: 3000,
      });
    });
  }
}
```

Finally, in order to enable the MatSnackBarModule, we need to include provide browser animations in our `app.config.ts` file.

#### app.config.ts

```typescript
import { provideAnimations } from '@angular/platform-browser/animations';
,
,
,
export const appConfig: ApplicationConfig = {
  providers: [
		provideAnimations(),
    .
    .
    .
```

You may need to install `@angular/animations` before this will work as well:

```bash
npm install @angular/animations@YOUR_ANGULAR_VERSION_NUMBER
```

To test, open two browser windows (including an incognito browser), one logged in as a user and the other as a different user. Make user 1 like user 2's blog and you should see a notification on user 2's browser.

# Any questions?
