---
title: Simplified Authorization with Policy Objects
date: 2024-05-05 19:25:00 +0000
categories: [Ruby on Rails, Layered Design for Ruby on Rails Applications]
tags: [authorization, policy objects, security, ruby on rails, application design]
---

Embedding authorization rules within the presentation layer muddles business logic.

Discover a cleaner, more efficient approach. Use policy objects - they enhance readability and maintainability in your app.

## Simplifying Authorization Enforcement with Policy Objects

Enforcing authorization is key to app design. But, where should these rules be? Embedding authorization rules in the presentation layer is not ideal. They should reflect business logic, not the delivery method. Yet, enforcing authorization must stay in the presentation layer. It must rely on rules set lower in the architecture.

Moving authorization rules into models may seem appealing, but it presents issues. Model methods lack context awareness, increasing complexity. Authorization rules don't belong in the domain layer, they belong in the authorized context. This requires a new abstraction at the services layer.

One approach is to use policy objects, encapsulating business rules, and reducing duplication. These objects correspond to resources in the application, such as domain model entities. For instance, in a library app, a `BookPolicy` class holds rules for book authorization.

```ruby
class BookPolicy
  attr_reader :user, :book

  def initialize(user, book)
    @user = user
    @book = book
  end

  def destroy?
    user.admin? || (user.librarian? && book.available?)
  end

  def edit?
    user.admin? || (user.librarian? && book.checked_out?)
  end

  def view?
    user.admin? || user.librarian? || book.public?
  end
end
```

We can use the policy object in the controller. It makes the code easier to read and maintain.

```ruby
class BooksController < ApplicationController
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
    unless BookPolicy.new(current_user, @book).destroy?
      redirect_to books_path, alert: "No access"
    end
  end
end
```

This approach clarifies the controller's responsibilities, streamlining the code and enhancing maintainability. We introduce an abstraction layer that uses policy objects. This improves readability and enables code reuse.

## Simplifying Authorization with Action Policy

Authorization in applications spans many layers. These layers include policy objects (services), enforcement points, and view filtering.

### Introducing Action Policy

> Check out this awesome gem: [action_policy](https://github.com/palkan/action_policy)
{: .prompt-tip }

Action Policy is a Ruby and Rails authorization framework which uses policy objects to define rules. It prioritizes performance, scalability, and developer experience, offering enhanced debugging and testing support.

### Conventions for Policy Classes

Naming conventions unify interfaces and reduce boilerplate. Under Rails conventions, name policy classes `<Model>Policy` for resource-based policies. Predicate methods like `#show?`, `#update?`, and `#destroy?` reflect authorization nature and simplify enforcement code.

### Streamlining Policy Classes

Designing policy classes around fundamental permission types like `#view? `and `#manage?` reduces redundancy. Aliases route actions to corresponding rules, enhancing readability and maintainability.

```ruby
class BookPolicy < ApplicationPolicy
  def view?
    true
  end

  def manage?
    return false unless user

    user.admin? || user.librarian? || record.user == user
  end
end
```

### Seamless Authorization Enforcement

Authorization checks in controllers are easier with **Action Policy**. They need just a single line of code for enforcement. The `#authorize!` The helper automates 3 things: policy identification, rule inference, and context invocation. It shows the power of abstraction.

```ruby
class BooksController < ApplicationController
  def destroy
    book = Book.find(params[:id])
    authorize! book
    book.destroy!
    redirect_to books_path, notice: "Removed"
  end
end
```

## Simplifying Authorization in Views

When it comes to UI components, filtering based on user permissions is essential. Leveraging policy objects for this purpose ensures a clean separation of concerns. For instance, with Action Policy, view templates can show controls based on permissions.


```erb
<li>
  <%= book.name %>
  <% if allowed_to?(:destroy?, book) %>
    <%= button_to "Delete", book, method: :delete %>
  <% end %>
</li>
```
The `#allowed_to?` helper simplifies authorization checks in views without raising exceptions.

### Handling Form Fields

Dealing with conditional form fields can be tricky. Using authorization models in views is OK. But, it can lead to redundant code and extra complexity. You can add custom rules to policy classes.

```ruby
class BookPolicy < ApplicationPolicy
  def search_by_isbn?() = user.librarian? || user.admin?
  def search_by_id?() = user.admin?
end
```

Then, in the view:

```erb
<%= form_for Books::SearchForm.new do |f| %>
  <%= f.text_field :q, placeholder: "Type a query.." %>
  <% if allowed_to?(:search_by_isbn?, Book) %>
    <%= f.text_field :isbn, placeholder: "ISNB" %>
  <% end %>
  <% if allowed_to?(:search_by_id?, Book) %>
    <%= f.text_field :book_id, placeholder: "Book ID" %>
  <% end %>
<% end %>
```

> Here we use Books::SearchForm class which is Form Object: [Check out more about Form Objects](https://patrogala.github.io/posts/form-objects/)
{: .prompt-info }

But, this approach can lead to bloated policy classes. An alternative is to define permitted form parameters within the policy class:

```rb
class BookPolicy < ApplicationPolicy
  def search_params
    return [] unless user

    params = []
    params << :isbn if user.admin? || user.librarian?
    params << :book_id if user.admin?
    params
  end
end
```

And in the view:

```erb
<%- policy = BookPolicy.new(user: current_user) -%>
<%= form_for Books::SearchForm.new do |f| %>
  <%= f.label :q, "Search Query" %>
  <%= f.text_field :q, placeholder: "Type a query.." %>

  <% if policy.search_params.include?(:isbn) %>
    <%= f.label :isbn, "ISBN" %>
    <%= f.text_field :isbn, placeholder: "ISBN" %>
  <% end %>

  <% if policy.search_params.include?(:book_id) %>
    <%= f.label :book_id, "Book ID" %>
    <%= f.text_field :book_id, placeholder: "Book ID" %>
  <% end %>
<% end %>
```

While these solutions help, they still introduce presentation logic into policy classes. Another way is to create standalone view policy objects. They would hold the authorization logic for UI elements. This approach ensures a cleaner separation of concerns and enhances maintainability.

## TL;DR

Authorization enforcement is vital yet often scattered.

Use policy objects to centralize and streamline rule management. With **Action Policy**, authorization checks become concise, improving code readability and maintainability.

By separating authorization concerns, you ensure a cleaner architecture and easier troubleshooting.

> Learn more about this topic from book: [Layered Design for Ruby on Rails Applications](https://www.packtpub.com/product/layered-design-for-ruby-on-rails-applications/9781801813785)
{: .prompt-info }


