# Image Uploading with Cloudinary

## Introduction

- We are going to be adding image attaching and uploading to beta_blogs

## Live Coding

### Adding Active Storage to a Ruby on Rails API

```bash
rails active_storage:install
```

```bash
rails db:migrate
```

Add the following to your blog model:

```ruby
class Blog < ApplicationRecord
  has_and_belongs_to_many :categories
  belongs_to :user
  validates :title, :content, presence: true

  has_one_attached :cover_image
end
```

Include an additional parameter `cover_image` in the `blog_params` method in the BlogsController:

```ruby
class BlogsController < ApplicationController
  before_action :authenticate_request
  .
  .
  .

  def blog_params
    params.permit(:title, :content, :cover_image)
  end
end
```

Use postman to create a new blog with a cover image.

- Click the body tab
- Choose `form-data`
- Login and get the token
- Make a new request and attach the token to the header
- Add a key for `title` and `content` and fill in the values
- Add a key of `cover_image` and change the type to `file`
- Choose a file to upload that is an image type
- Click `Send`

Check if the image is uploaded by going to the rails console:

```bash
Blog.last.cover_image.attached?
```

### Send Image URL in Response

Let's go ahead and include the image url when we create a new blog.

```ruby
class Blog < ApplicationRecord
  include Rails.application.routes.url_helpers
  .
  .
  .

  def cover_image_url
    rails_blob_url(self.cover_image, only_path: false) if self.cover_image.attached?
  end
end
```

Include the url_helpers to use the `rails_blob_url` method.

`rails_blob_url` is a method that generates a URL for a blob. `only_path: false` will generate a relative URL.

Since we include the url helpers, we will need to specify a host in the `config/environments/development.rb` file.

```ruby
Rails.application.configure do
  # Settings specified here will take precedence over those in config/application.rb.
  Rails.application.routes.default_url_options[:host] = "localhost:3000"
  .
  .
  .
```

Now we can include the `cover_image_url` in the response.

We will use `cover_image_url` for our blueprinter.

```ruby
# frozen_string_literal: true

class BlogBlueprint < Blueprinter::Base
  identifier :id

  fields :title, :content, :cover_image_url

  .
  .
  .
end
```

Test by creating a new blog and check if the image url is included in the response.

To check if the image exists you can visit the url in the browser.

Here's an example:

```bash
http://localhost:3000/rails/active_storage/blobs/redirect/eyJfcmFpbHMiOnsibWVzc2FnZSI6IkJBaHBHUT09IiwiZXhwIjpudWxsLCJwdXIiOiJibG9iX2lkIn19--8eb6f1767b264526edde29897f3e11ef17f93559/download%20(2).png
```

### Uploading Images from the Frontend

Let's adjust our blog model to include `cover_image_url`

```typescript
import { User } from './user';

export class Blog {
  id!: number;
  title: string = '';
  content: string = '';
  cover_image_url: string = '';
  user_id?: number;
  user?: User;
}
```

Let's adjust our blog component to include a file input for the cover image.

```html
<div>
  <label for="cover_Image">Cover Image</label>
  <input type="file" (change)="onFileSelected($event)" id="coverImage" />
</div>
```

```typescript
.
.
.
selectedFile: File | null = null;

constructor(private blogService: BlogService, private router: Router) {}

onFileSelected(event: any) {
  if (event.target.files && event.target.files[0]) {
    this.selectedFile = event.target.files[0];
  }
}
```

Adjust the `onSubmit` method to include the cover image.

We have to use `FormData` to send the file to the server as a multipart form. This is because we are sending a file. We can't do it with a JSON object.

```typescript
onSubmit() {
  if (this.blogForm.valid && this.selectedFile) {
    const formData = new FormData();
    formData.append('title', this.blogForm.get('title')!.value!);
    formData.append('content', this.blogForm.get('content')!.value!);
    formData.append('cover_image', this.selectedFile, this.selectedFile.name);

    this.blogService.createBlog(formData).subscribe({
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

Adjust the blog list component to display the cover image.

```html
<div class="blog-container">
  @for (blog of blogs; track blog.id) {
  <div class="blog-preview-container">
    <div class="blog-preview-content">
      <h2>{{ blog.title }}</h2>
      <p>{{ blog.user?.username }}</p>
      <p>{{ blog.content }}</p>
    </div>
    <img
      [src]="
        blog.cover_image_url ||
        'https://assets.bitdegree.org/online-learning-platforms/storage/media/2018/08/how-to-become-a-programmer.jpg'
      "
      alt="blog image"
      class="blog-image"
    />
  </div>
  }
</div>
```

### Adding Cloudinary to the Rails API

Please sign up for a free account.

Once done, install the `cloudinary` gem by adding it to your `Gemfile`:

```ruby
# Gemfile
group :production do
  gem 'pg'
  gem 'cloudinary'
end
```

Then run:

```bash
bundle install
```

Next, you will need to configure the `cloudinary gem`. You can do this by adding the following to your `config/environments/production.rb` file:

#### config/environments/production.rb

```ruby
require "active_support/core_ext/integer/time"

Rails.application.configure do
  # Settings specified here will take precedence over those in config/application.rb.
.
.
.
  config.active_storage.service = :cloudinary

.
.
.
```

Replace `config.active_storage.service = :local` with `config.active_storage.service = :cloudinary`.

This will configure Active Storage to use Cloudinary as the storage service in production. In development, we will use local storage.

We then need to add the `cloudinary` configuration to `storage.yml`. The `storage.yml` file is used to configure the storage services that are used by Active Storage.

```yaml
test:
  service: Disk
  root: <%= Rails.root.join("tmp/storage") %>

local:
  service: Disk
  root: <%= Rails.root.join("storage") %>
```

We are going to need to add the `cloud_name`, `api_key`, and `api_secret` to the `storage.yml` file. Let's go to our Cloudinary account to get these details.

Navigate to your Cloudinary dashboard and you should see your `cloud_name`, `api_key`, and `api_secret`. Copy these details and add them to the `storage.yml` file.

Include the `cloud_name`, `api_key`, and `api_secret` in the credentials files.

```bash
EDITOR="code --wait" bin/rails credentials:edit
```

Here we will store sensitive information like the `cloud_name`, `api_key`, and `api_secret` in the `credentials.yml.enc` file.

```yaml
# aws:
#   access_key_id: 123
#   secret_access_key: 345

cloudinary:
  cloud_name: 'djlttsuf3'
  api_key: '1165244111841913'
  api_secret: 'pJPGYiJv3fhIC1ezruSQWjbkOJjk'

# Used as the base secret for all MessageVerifiers in Rails, including the one protecting cookies.
secret_key_base: 908a3dca583e7754b741df137fb9a3b2c756c47448a349eacadfc4202c38f8682e5e3753f0dc858c22f7536e5ead6934ac5f8be3c48ca62f268a348ed409fdae
```

Make sure your cloudinary input is formatted correctly or you will get an error.

Close the file so it can be saved.

Now you can access the `cloudinary` configuration in your `storage.yml` file.

```yaml
test:
  service: Disk
  root: <%= Rails.root.join("tmp/storage") %>

local:
  service: Disk
  root: <%= Rails.root.join("storage") %>

cloudinary:
  service: Cloudinary
  cloud_name: <%= Rails.application.credentials.cloudinary[:cloud_name] %>
  api_key: <%= Rails.application.credentials.cloudinary[:api_key] %>
  api_secret: <%= Rails.application.credentials.cloudinary[:api_secret] %>
```

Before testing this in production, let's be sure to add the host from the url of our API to the `config/environments/production.rb` file.

```bash
Rails.application.routes.default_url_options[:host] = "https://betablogclassname.onrender.com/"
```

Let's test this in production by opening the front end and creating a new blog.

# Any questions?
