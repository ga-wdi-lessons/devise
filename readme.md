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
$ rails db:drop db:create db:migrate db:seed
$ rails s
```

## Devise

Devise is a gem that simplifies implementing user authentication. - https://github.com/plataformatec/devise#starting-with-rails

While it's interesting to know what's happening under the hood when authenticating a user, it's very unlikely
you'll implement your own authentication mechanism in any future project. Devise is a well-tested open source
authentication solution for rails.

In yesterday's lesson, passwords were stored in the database in plaintext.

<details>
<summary>Why might storing passwords in the database be a bad idea?</summary>

- We might get hacked
- Some users have the same password for every service (not me)
- http://plaintextoffenders.com/faq/devs

</details>

<details>
<summary>What are the alternatives to storing passwords in plaintext?</summary>

- Hashing
- Encryption
</details>

### You do: SO Hunt

Read this question and answer on Stack Overflow - http://stackoverflow.com/questions/4948322/fundamental-difference-between-hashing-and-encryption-algorithms

Prepare to answer the following questions:

- What is the difference between encryption and hashing?
- When would you choose one over the other?
- Which one should be used for passwords?

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

<details>
<summary>When working with devise, what do you think might show up inside a flash message?</summary>

<ul>
<li>Whether the user has signed in or out successfully</li>
<li>Whether the password is correct/incorrect</li>
<li>Whether the username is taken</li>
</ul>
</details>

### Commit!

Seriously, commit now. It will be easy to fix any devise issues if you have a commit you can go back to.

### Generate User model

```
$ rails g devise User
```

>If you have an existing User model, Devise should build on the functionality. But, if you're getting migration errors, it may be easier to recreate the model using Devise and add your functionality back.

### You do: Check it out!

- What did the previous command generate?
- What files did running the previous command change?
- What is the output of `rails routes`

### Run migrations

```
$ rails db:migrate
$ rails s
```

### Link to Sign Up page

```erb
<!-- app/views/layouts/application.html.erb -->
<%= link_to "Sign Up", new_user_registration_path %>
```

>`new_user_registration_path` provided by Devise - from `$ rails routes`

### Link to Sign Up only if not signed in

```erb
<!-- app/views/layouts/application.html.erb -->
<% if !current_user %>
  <%= link_to "Sign Up", new_user_registration_path %>
<% end %>
```
>current_user is a helper method provided by Devise to get the user from the session. Another helper method you can use to check if a user is signed in is the self-explanatory user_signed_in?.

### You do (5 mins):

If the user is logged in, show link to log out, including the user's email.

Otherwise, show both a link to sign up and sign in.

<details>
<summary>Solution</summary>

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
</details>

### You do: Associate Posts with a User (10 mins)

Modify your `User` and `Post` classes so that a user has many
posts and a post belongs to a user.

Create a few seeds to verify you did this part correctly.


<details>
<summary>Solution</summary>

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

</details>

## We do: Update the Controller

```rb
# app/controllers/posts_controller

def create
  @post = Post.create!(post_params.merge(user: current_user))
  redirect_to post_path(@post)
end
```

>notice: no need to update strong_params - this could be bad, actually!

[`merge`](https://ruby-doc.org/core-2.2.0/Hash.html#method-i-merge) is a built-in ruby method for combining two hashes.

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

## You do: Prevent Users from editing someone else's post.

## You do: Associate users with Comments

1. Create a new migration to add a user_id column to comments.
2. Associate the `current_user` with newly created posts.
3. Prevent User's from editing / deleting other's comments.

## Customizing Views

Devise has generated several views for us, but they are not currently visible in our
codebase.

If you want to customize them, generate the views with `rails g devise:views`

The possibilities are endless!

### Styling views

Whether you generated the views or not, you can style the forms the same way.

Identify the selectors you'd use to target the individual form elements, and add
styles in `app/assets/stylesheets/application.css`

## If there's time...

Try implementing an authorization solution on top of devise - https://github.com/ga-wdi-lessons/cancancan
