---
title: Form Objects in Rails - Why Models Shouldn't Touch UI
date: 2024-04-21 15:33:55 +0000
categories: [Ruby on Rails, Layered Design for Ruby on Rails Applications]
tags: [ruby, rails, web, layered design, models, active record, form object, ui]
---

In Ruby on Rails development, elegant code and efficient architecture are key. But, one often overlooks a critical thing: the separation of concerns.

Models are a fundamental part of Rails. They should be oblivious to user interface (UI) concerns. This idea might sound radical to some, but it’s a game changer. It makes for a clean, scalable, and organized codebase.

This change in view isn't just a recommendation. It's a necessity for modern Rails apps. They aim for lasting efficiency and easy maintenance.

In this discussion, we'll also cover the use of form objects to change how we handle data in Rails. This approach avoids traditional pitfalls like cluttered models.

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->

> Check out the Repository Design Pattern at [Rails Data Management with the Repository Pattern](https://patrogala.github.io/posts/repositories/)
{: .prompt-info }

<!-- markdownlint-restore -->

We'll start with a clear split of duties at the core of your app's architecture.

## Should Models Know About the UI? No!

In a well-structured Rails app, models should not handle UI tasks. Why? Because models are meant to manage data, not how it's presented or interacted with. This concept is important in maintaining clean and organized code.

### Example: Building a Rails App

Imagine we're starting a new app. Using Rails' `scaffold` command, we can set up a basic structure:

```bash
$ bin/rails g scaffold user name:string email:string
```

It makes: database migrations, model files, a controller for user actions, and views for user forms.

### Problem with Model-UI Mix

While this setup is quick, it often leads us to mix UI logic with model logic. For example, when we add features like user invitations or registrations.

The original setup isn't enough. We might end up adjusting the model to handle different forms and actions, which gets messy.

### Better Approach: Use Form Objects

Instead of cramming everything into the model or controller, we can use form objects. Form objects handle data submission and validation for specific user interactions. They keep UI logic out of the model, making the code cleaner and more organized.

### Implementing a Form Object

Here’s a simple form object for inviting a user:

```ruby
class UserInvitationForm
  attr_reader :user, :send_copy, :sender

  def initialize(params, send_copy: false, sender: nil)
    @user = User.new(params)
    @send_copy = send_copy
    @sender = sender
  end

  def save
    return false unless valid?
    user.save!
    send_notifications
    true
  end

  private

  def valid?
    user.email.present?
  end

  def send_notifications
    UserMailer.invite(user).deliver_later
    UserMailer.invite_copy(sender, user).deliver_later if send_copy
  end
end
```

This form object manages user invitations. It checks for valid input and handles email notifications. This keeps the model focused only on data management.

## Key Tasks for Form Objects in Rails

Form objects in Rails are essential for handling user interactions smoothly. They primarily manage three tasks:

1. **Filtering and Validating Input**: Form objects ensure that all user input is valid. They do this before processing it.

2. **Managing Actions on Submission**: When a form is submitted, form objects trigger actions based on the submission.

3. **Working with Rails Helpers**: Form objects need to work well with Rails' view helpers to be effective in forms.

### Example: Simplified Form Object

Let's create a basic form object using Rails’ ActiveModel for attributes and validations:

```ruby
class SignupForm
  include ActiveModel::Model
  attr_accessor :email, :should_send_newsletter

  validates :email, presence: true, format: { with: URI::MailTo::EMAIL_REGEXP }

  def submit
    return false unless valid?
    user = User.create(email: email)
    send_newsletter(user) if should_send_newsletter
    true
  end

  private

  def send_newsletter(user)
    NewsletterMailer.send_to(user).deliver_later
  end
end
```

This form object handles user sign-ups. It checks the email and, optionally, sends a newsletter. The submit method centralizes the form's logic. It ensures actions are taken only after successful validation.

### Using Form Objects with Views

When integrating form objects with Rails views, they behave much like models. Here’s how you might use our SignupForm in a view:

```erb
<%= form_for(@signup_form, url: signup_path) do |form| %>
  <%= form.label :email %>
  <%= form.email_field :email %>
  <%= form.label :send_newsletter, "Subscribe to newsletter?" %>
  <%= form.check_box :send_newsletter %>
  <%= form.submit "Sign Up" %>
<% end %>
```

## Conclusion

Using form objects in Ruby on Rails applications separates concerns. This underscores a cleaner, more maintainable way to manage user interactions.

Form objects decouple the UI logic from the models. They keep the models pure, focused only on data integrity. They also enhance the scalability and flexibility of the application.

This approach follows best practices in software architecture. It promotes a more organized codebase. It also makes testing and maintenance easier.

Using form objects well makes complex user interactions and validations easier to handle.

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->

> Learn more about this topic from book: [Layered Design for Ruby on Rails Applications](https://www.packtpub.com/product/layered-design-for-ruby-on-rails-applications/9781801813785)
{: .prompt-info }

<!-- markdownlint-restore -->

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->

> Check out the Form Objects Design Pattern at [Nimble](https://nimblehq.co/blog/lets-play-design-patterns-form-objects)
{: .prompt-info }

<!-- markdownlint-restore -->
