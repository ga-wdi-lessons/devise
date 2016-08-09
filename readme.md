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
If you have an existing User model, Devise should build on the functionality. But, if you're getting migration errors, it may be easier to recreate the model using Devise and add your functionality back.

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

>btw. I got this path, provided by Devise, from `$ rake routes`

### Link to Sign Up only if not signed in

```erb
<!-- app/views/layouts/application.html.erb -->
<% if !current_user %>
  <%= link_to "Sign Up", new_user_registration_path %>
<% end %>
```
>current_user is a helper method provided by Devise to get the user from the session. Another helper method you can use to check if a user is signed in is the self-explanatory user_signed_in?.

### I do:

If the user is logged in, show link to log out, including the user's email.

Otherwise, show both a link to sign up and sign in.


```erb
<!-- app/views/layouts/application.html.erb -->
<% if !current_user %>
  <%= link_to "Sign up", new_user_registration_path %>
  <%= link_to "Sign in", new_user_session_path %>
<% else %>
  <%= link_to "Sign out", destroy_user_session_path, :method => :delete %>
    <%= current_user.email %>
<% end %>
```
>The delete method needs to be specified. Get is the default for the link_to helper.

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
# app/controllers/posts_controller

def create
  @post = Post.create!(post_params.merge(user: current_user))
  redirect_to post_path(@post)
end
```

>notice: no need to update strong_params - this could be bad, actually!

## Limiting User Abilities

```rb
# app/controllers/posts_controller

def destroy
  @post = Post.find(params[:id])
  if @post.user == current_user
    @post.destroy
  else
    flash[:alert] = "Only the author of the post can delete"
  end
  redirect_to posts_path
end
```

## Bonus!

If you need more fine-grained control over user permissions, check out [CanCanCan](https://github.com/CanCanCommunity/cancancan)
