# Devise

## Learning Objectives

- Implement user authentication
- `merge` two hashes to associate user
- Use devise helper methods

## Setup

```
$ git clone https://github.com/ga-wdi-exercises/scribble.git
$ cd scribble
$ git checkout pre-devise
$ rake db:create db:migrate db:seed
```

## Devise

Devise is a gem that simplifies implementing user authentication. - https://github.com/plataformatec/devise#starting-with-rails

### Install Dependencies

```
# Gemfile

gem "devise"
```

```
$ bundle install
$ rails g devise:install
```

From the output of the previous command, let's make sure we do #3 only:

```
 3. Ensure you have flash messages in app/views/layouts/application.html.erb.

For example:

<p class="notice"><%= notice %></p>
<p class="alert"><%= alert %></p>
```

### Commit!

Seriously, commit now. It will be easy to fix any devise issues if you have a commit you can go back to.

### Generate User model

```
$ rails g devise User
```

> But what if I already have a User model?

That's ok!

### Run migrations

```
$ rake db:migrate
$ rails s
```

### Link to Sign Up page

```erb
<!-- app/views/layouts/application.html.erb -->
<%= link_to "Sign Up", new_user_registration_path %>
```

>btw. I got this path from `$ rake routes`

### Link to Sign Up only if not signed in

```erb
<!-- app/views/layouts/application.html.erb -->
<% if !current_user %>
  <%= link_to "Sign Up", new_user_registration_path %>
<% end %>
```

### You do:

If the user is logged in, show link to log out, including the user's email.

Otherwise, show both a link to sign up and sign in.

### Associating Posts with a User

```rb
# app/models/user

has_many :posts
```

```rb
# app/models/post

belongs_to :user
```

```
$ rails g migration add_users_to_posts user:references
$ rake db:migrate
```

```rb
app/controllers/posts_controller

def create
  Post.create(post_params.merge(user: current_user))
end
```