---
title: How to improve Rails with presenters and decorators?
date: 2024-05-01 10:04:00 +0000
categories: [Ruby on Rails, Layered Design for Ruby on Rails Applications]
tags: [ruby, rails, web, layered design, models, active record, presentation layer, presenters]
---

When coding in Rails, do you ever find your models burdened with tasks they shouldn't be handling?

It is a common pitfall. Models are designed purely for data handling. But, they end up bloated with view-specific formatting code.

This confuses what models are for. It also makes your app harder to maintain. The solution lies in a sleek, yet often overlooked, feature of Rails: **Presenters**.

This guide delves into how to use presenters and decorators. They can greatly clean up your code. They make sure models and views stick to their proper roles and work well together.

## Using Presenters to Separate Models from Views in Rails

In Rails, models and views have a simple relationship. Views display data from models. However, sometimes models end up handling tasks meant for views. They format data for display. This mixes responsibilities and can clutter the model with non-essential code.

### Problem with Mixing View Logic in Models

For example, a User model might include a method to format a name in a shortened style:

```ruby
class User < ApplicationRecord
  def short_name
    name.split(" ").map(&:first).join
  end
end
```

This coding approach leads to models. They directly manipulate data presentation, which is not ideal.

## What about Rails helpers?

Rails has built-in view helpers, which are Ruby methods mainly used by templates. When you add a new resource in a Rails app, it creates a helper module for you. You use it to put view-related logic.

```ruby
module UsersHelper
  def user_short_name(user)
    user.name.split(" ").map(&:first).join
  end
end
```

Helpers are global, meaning they are accessible throughout your app. However, you can turn off this feature and manage helpers more closely if needed.

They are useful for code reuse in views. But, managing many helpers can be hard. This is because there's no easy way to organize them. This is due to their global nature and lack of advanced object-oriented features. For more complex view logic, it's better to use other abstractions.


## Presenters and Decorators

Presenters and decorators help manage how objects are shown in specific views. Here’s a simplified explanation:

**Presenters**:
Presenters wrap models to shape how data is displayed in views. They act as a middle layer, making the model-view interaction cleaner.

Create a `UserPresenter` to simplify user display in a view:

```ruby
class UserPresenter
  def initialize(user)
    @user = user
  end

  def short_name
    @user.name.split(" ").map(&:first).join
  end
end
```

Use it in a view like this:

```html
<div id="user-<%= @user.id %>">
  <%= link_to UserPresenter.new(@user).short_name, @user %>
</div>
```

**Decorators**:
Decorators extend objects by adding new behaviors dynamically. This doesn’t change the original object but enhances it for specific contexts.

**Example**:
Create a UserDecorator for additional user methods:

```ruby
class UserDecorator < SimpleDelegator
  def full_name
    __getobj__.name.titleize
  end
end
```

> Learn more about SimpleDelegator class: [ruby-doc.org](https://ruby-doc.org/stdlib-2.5.1/libdoc/delegate/rdoc/SimpleDelegator.html)
{: .prompt-info }

Use this decorator to enhance user presentation without altering the user model.

**Open vs. Closed Presenters:**
- **Open presenters**: Allow all methods of the original object to be available.
- **Closed presenters**: Limit the methods to what’s explicitly defined.

Whether you choose open or closed depends on your project's needs. Open presenters might start off easier. They allow seamless access. Closed presenters provide tighter control over what's exposed to the view. They enhance security and cleanliness.

## Presenters Layer in Rails App

This turns the presenter pattern into an abstraction layer. It involves setting rules and organizing code. Here's a simplified breakdown:

**Naming and Code Organization:**
In Rails, using naming conventions reduces confusion and extra coding. For example, adding "Presenter" at the end of a class name clearly shows its role. Typically, presenter classes are stored in the `app/presenters` directory.

**Helper for Presenters:**
You can create a helper method, like present, to simplify using presenters in views:

```ruby
module ApplicationHelper
  def present(obj, with: nil)
    presenter_class = with || "#{obj.class.name}Presenter".constantize
    presenter_class.new(obj)
  end
end
```

This helper automatically chooses the right presenter for an object. In the view, you'd use it like this:

```html
<div id="user-<%= user.id %>">
  <%= link_to present(user).short_name, user %>
</div>
```

**Keynote Gem:**
Keynote makes creating and using presenters easier. It also caches presenters to reuse them. Here’s how you might use it:

> Check out this awesome gem: [rf- keynote](https://github.com/rf-/keynote)
{: .prompt-tip }

```ruby
class UserPresenter < Keynote::Presenter
  presents :user

  def short_name
    user.name.split(" ").map(&:first).join
  end
end
```

In a view, use the Keynote helper `k`:

```html
<div>
  <span><%= k(user).short_name %></span>
</div>
```

**Integrating View Helpers:**
You can make presenters more powerful. You can do this by letting them use Rails view helpers directly. For instance:

```rb
class PostPresenter < Keynote::Presenter
  presents :post
  def status_icon
    content_tag(:i, "", class: "fa #{post.draft? ? 'fa-hourglass-start' : 'fa-calendar-check'}")
  end
end
```

And in your view:

```html
<div>
  <%= k(post).status_icon %>
  <span><%= post.title %></span>
</div>
```

Keep presenters within the presentation layer. Don’t let them seep into lower layers. Also, don't let them replace real objects outside of views. This will prevent unexpected behavior in other parts of your application.

Use presenters mainly in views. They make things clearer and more organized.

> Learn more about this topic from book: [Layered Design for Ruby on Rails Applications](https://www.packtpub.com/product/layered-design-for-ruby-on-rails-applications/9781801813785)
{: .prompt-info }

## Conclusion: Rails Views with Presenters and Decorators

In short, adding presenters and decorators to a Rails application is a smart move.

It helps to separate models from views. This follows the ideas of clean architecture. Delegating the presentation responsibility to dedicated classes helps Rails developers. It lets them keep lean models and tidy views.

