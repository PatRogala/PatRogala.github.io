---
title: Secure Your Web App with Authentication & Authorization
date: 2024-05-04 09:26:00 +0000
categories: [Ruby on Rails, Layered Design for Ruby on Rails Applications]
tags: [ruby, rails, web, layered design, models, authorization, authentication, web security]
---

**Ever wondered how digital systems verify your identity and decide what you can do?** That's the magic of authentication and authorization at work. It's like checking an ID at the door and then deciding which rooms you're allowed to enter.

## Understanding Authentication and Authorization

Authentication is about verifying who's making a request. Once verified, we issue a token for future requests. Authorization determines what actions a user can perform. It checks if the identified user has the right permissions to do something.

### Example in Action

Let's say we have a controller action for deleting a post. We use authentication to check the user's identity. We use authorization to see if they're allowed to delete the post.

```ruby
class PostsController < ApplicationController
  before_action :authenticate! # authentication
  before_action :set_post, only: [:destroy]
  before_action :authorize_user!, only: [:destroy] # authorization

  def destroy
    @post.destroy!
    redirect_to posts_path, notice: "Post has been deleted"
  rescue ActiveRecord::RecordNotDestroyed
    redirect_to posts_path, alert: "Post could not be deleted"
  end

  private

  def set_post
    @post = Post.find(params[:id])
  end

  # Here we are checking if the user is the owner of the post
  def authorize_user!
    redirect_to posts_path, alert: "You are not allowed to delete this post" unless @post.user_id == current_user.id
  end
end

```

### Securing Different Layers

![Desktop View](/assets/images/secure-layers-of-web-app.webp){: width="972" height="589" }
_Secure Layers of Web App_

Securing a web app involves protecting various layers. We also need to consider bussiness constraints and data validation. These are beyond just authorization and authentication.

### Bussiness Constraints
These are rules that apply regardless of the user. For instance, this could mean limiting the number of actions a user can do per day.

### Data Validation
At the domain level, we ensure data consistency through validations. This ensures that the data meets specific criteria before processing it. Like checking if post title is present and shorter than 100 characters.

## Simplifying Authorization: Domain-less Models
When it comes to authorization, simplicity can be key. In some apps, like note systems, you don't always need complex authorization. Take this example:

```ruby
class NotesController < ApplicationController
  before_action :authenticate! # authentication
  before_action :set_note, only: [:destroy]
  before_action :authorize_user!, only: [:destroy]

  def destroy
    @note.destroy!
    redirect_to notes_path, notice: "Note has been deleted"
  rescue ActiveRecord::RecordNotDestroyed
    redirect_to notes_path, alert: "Note could not be deleted"
  end

  private

  def set_note
    @note = Note.find(params[:id])
  end

  # We are checking if logged in user is the owner of the note
  def authorize_user!
    redirect_to notes_path, alert: "Forbidden" unless @note.user_id == current_user.id
  end
end

```

Here (just like in previous example), we're simply checking if the user owns the note before allowing deletion. But what if we need to differentiate between regular users and admins?

### Role-based Access Control (RBAC)

![Desktop View](/assets/images/rbac.webp){: width="972" height="589" }
_Role Based Access Control_

RBAC is a common approach where each user has a role granting them specific privileges. For example, in a library app, we might have administrators. We also have librarians and regular users.

```ruby
class User < ApplicationRecord
  enum role: { regular: 0, admin: 1, librarian: 2 }
end

```

With RBAC, we can easily check permissions in controllers:

```ruby
class BooksController < ApplicationController
  before_action :authenticate! # authentication
  before_action :require_access, only: [:new, :create, :edit, :update, :destroy]

  private

  def require_access
    return if current_user&.librarian? || current_user&.admin?
    redirect_to books_path, alert: "No access"
  end
end

```

### Attribute-based Access Control (ABAC)

![Desktop View](/assets/images/abac.webp){: width="972" height="589" }
_Attribute-based Access Control_

ABAC extends RBAC by considering attributes of objects and subjects for access decisions. For instance, in our library example, we might restrict librarians. They could only delete books from their departments.

```ruby
class BooksController < ApplicationController
  before_action :authenticate! # authentication
  before_action :set_book, only: [:destroy]
  before_action :authorize_user!, only: [:destroy]

  def destroy
    @book.destroy!
    redirect_to books_path, notice: "Removed"
  rescue ActiveRecord::RecordNotDestroyed
    redirect_to books_path, alert: "Book could not be deleted"
  end

  private

  def set_book
    @book = Book.find(params[:id])
  end

  def authorize_user!
    return if current_user.permission?(:manage_all_books) || (
      current_user.permission?(:manage_books) &&
      @book.dept == current_user.dept
    )

    redirect_to books_path, alert: "No access"
  end
end

```

ABAC offers more flexibility but requires careful consideration of authorization rules and enforcement.

## TL;DR: Understanding Authentication and Authorization

Authentication verifies the user's identity. It issues tokens for future requests. Authorization determines user actions. Securing web apps involves system constraints and data validation. Simplifying authorization can mean using domain-less models. It can also mean using Role-based Access Control (RBAC) or Attribute-based Access Control (ABAC). These give more detailed permissions.

> Learn more about this topic from book: [Layered Design for Ruby on Rails Applications](https://www.packtpub.com/product/layered-design-for-ruby-on-rails-applications/9781801813785)
{: .prompt-info }


