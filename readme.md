# Devise

## Learning Objectives

- Implement user authentication
- `merge` two hashes to associate the user model
- Use Devise helper methods
- Explain the difference between encryption and hashing algorithms

## Setup

```
$ git clone https://github.com/ga-wdi-exercises/scribble.git
$ cd scribble
$ git checkout pre-devise
$ rails db:drop db:create db:migrate db:seed
$ rails s
```

## Devise

[Devise](https://github.com/plataformatec/devise#starting-with-rails) is a gem that simplifies implementing user authentication.

While it's interesting to know what's happening under the hood when authenticating a user, it's very unlikely you'll implement your own authentication mechanism in any future project. Devise is a well-tested open source authentication solution for rails.

In yesterday's lesson, passwords were stored in the database in plaintext.

<details>
  <summary><strong>Why might storing passwords in the database be a bad idea?</strong></summary>

  - We might get hacked
  - Some users have the same password for every service (not me)
  - [Plain Text Offenders](http://plaintextoffenders.com/faq/devs)

</details>

<details>
  <summary><strong>What are the alternatives to storing passwords in plaintext?</strong></summary>

  - Hashing
  - Encryption

</details>

### You Do: Stack Overflow Hunt

Read [this question](http://stackoverflow.com/questions/4948322/fundamental-difference-between-hashing-and-encryption-algorithms) and answer on Stack Overflow.

Prepare to answer the following questions...
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

<!-- AM: Show what the output is -->

From the output of the previous command, let's make sure we do #3 only...

```
 3. Ensure you have flash messages in app/views/layouts/application.html.erb.

For example:

<p class="notice"><%= notice %></p>
<p class="alert"><%= alert %></p>
```

<details>
  <summary>When working with Devise, what do you think might show up inside a flash message?</summary>

  - Whether the user has signed in or out successfully
  - Whether the password is correct/incorrect
  - Whether the username is taken

</details>

### Commit!

Seriously, commit now. It will be easy to fix any devise issues if you have a commit you can go back to.

### Generate User model

```
$ rails g devise User
```

> If you have an existing User model, Devise should build on the functionality. But, if you're getting migration errors, it may be easier to recreate the model using Devise and add your functionality back.

### You Do: Check It Out!

- What did the previous command generate?
- What files did running the previous command change?
- What is the output of `rails routes`?

### Run Migrations

```
$ rails db:migrate
$ rails s
```

### Link to "Sign Up" Page

```erb
<!-- app/views/layouts/application.html.erb -->
<%= link_to "Sign Up", new_user_registration_path %>
```

>`new_user_registration_path` provided by Devise - from `$ rails routes`

### Link to "Sign Up" Only If Not Signed In

```erb
<!-- app/views/layouts/application.html.erb -->
<% if !current_user %>
  <%= link_to "Sign Up", new_user_registration_path %>
<% end %>
```
> `current_user` is a helper method provided by Devise to get the user from the session. Another helper method you can use to check if a user is signed in is the self-explanatory user_signed_in?.

### You Do

<!-- AM: Show example of this -->

If the user is logged in, show a link to log out. The link should display the user's email right next to it.

Otherwise, show both a link to sign up and sign in.

<details>
  <summary><strong>Solution...</strong></summary>

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

  > The delete method needs to be specified. `GET` is the default for the `link_to` helper.

</details>

### You Do: Associate Posts with a User

Modify your `User` and `Post` classes so that...
- A user has many posts
- A post belongs to a user.

Create a few seeds to verify you did this part correctly.

<details>
  <summary><strong>Solution...</strong></summary>

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

## We Do: Update the Controller

```rb
# app/controllers/posts_controller

def create
  @post = Post.create!(post_params.merge(user: current_user))
  redirect_to post_path(@post)
end
```

> You do not need to modify `strong_params`

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

## You Do: Prevent Users From Editing Someone Else's Post

## You Do: Associate Users With Comments

1. Create a new migration to add a user_id column to comments
- Associate the `current_user` with newly created posts
- Prevent User's from editing / deleting other's comments

## Customizing Views

Devise has generated several views for us but they are not currently visible in our
codebase.

If you want to customize them, generate the views with `rails g devise:views`

### Styling Views

Whether you generated the views or not, you can style the forms the same way.

Identify the selectors you would use to target the individual form elements, and add
styles in `app/assets/stylesheets/application.css`

## If There's Time...

Try implementing an authorization solution on top of Devise using [CanCanCan](https://github.com/ga-wdi-lessons/cancancan)
