---
title: Upgrade Your Rails UI - Introducing View Components
date: 2024-05-05 19:25:00 +0000
categories: [Ruby on Rails, Layered Design for Ruby on Rails Applications]
tags: [rails, ui, view components, rendering, parameters, development, web development]
---

**Ever faced rendering failures in Rails due to missing parameters?**

Rails templates are vital for UI. But, they often cause headaches, this is due to their reliance on parameters and hidden dependencies. But fear not! We're exploring a solution: View Components.

We'll change Rails templates into self-contained components. They will be like ones in React or Vue. This will make interfaces clearer, increase reusability, and ease maintenance.

## Rails Templates for bulding UI

In Rails, templates are a crucial part of building user interfaces (UIs). Templates are unlike models and controllers. They are written in markup languages like HTML or Haml. This makes them readable by humans and machines. But, it lacks the structure of a programming language.

Consider a scenario: We're rendering a view that shows a student's course quiz result. We define the UI in a partial template:

```erb
<div>
  <header>
    <h3><%= course.title %></h3>
  </header>
  <p>
    <i class="fa fa-user"></i>
    <%= link_to user.username, user %>
  </p>
  <div>
    <label>Score</label>
    <span><%= user_lesson.score %> / <%= lesson.score %>
      (<%= l(result.created_at) %>)</span>
  </div>
</div>
```

The challenge arises when determining the required arguments for rendering this partial. Without the necessary parameters, rendering fails, highlighting the need for clear interfaces:

```ruby
render partial: "lessons/student_result", locals: {}
#=> undefined local variable or method `course'…
```

This issue is missing parameters. It can get worse with conditional logic in templates. This complexity makes maintenance harder and risks bugs in production.

Also, connecting templates with controller state via instance variables creates hidden dependencies. This makes maintenance hard.

### Example of Strict Locals

Rails offers techniques like strict locals and linting tools to mitigate these issues. But, they're not complete fixes.

> Strict Locals was added in Rails 7.1
{: .prompt-danger }

```erb
<%# app/views/lessons/_student_result.html.erb %>

<%# locals: (course:, user:, user_lesson:, lesson:) -%>

# Code from previous example
```

> Learn more about [Strict Locals](https://edgeguides.rubyonrails.org/action_view_overview.html#strict-locals)
{: .prompt-info }

> You can use Shopify' [erb-lint](https://github.com/Shopify/erb-lint) as your erb linter
{: .prompt-tip }

Helpers aid in making Rails-aware HTML. But, relying only on partials and helpers for UI has challenges. This is especially true with changing design systems.

To meet these challenges, we need a better way to represent UI elements. It should support the evolution of design systems, promote reusability, and simplify maintenance. This requires better partials, more helpful helpers, or new abstractions.

## Views Components for better management

Breaking down views into self-contained parts has been a game-changer in frontend development. Libraries like React and Vue have led the way. Can we bring this approach to Rails? Sure we can!

### Turning Partials and Helpers into Components

We need to change partials and helpers into components. We must make sure they are isolated. They must be self-contained and have clear interfaces. You can do this by using Ruby classes. They represent components and split logic from presentation.

### Components as Ruby Objects

For instance, consider a search box component. We define a Ruby class for it, encapsulating its logic:

```ruby
class SearchBoxComponent
  attr_reader :action, :text

  def initialize(action:, text: "Search", button: false)
    @action = action
    @text = text
    @button = button
  end

  def button?
    @button
  end
end
```

### HTML for Components

Next, we make an HTML template for the component.

```erb
<%= form_with(url: action) do |f| %>
  <i class="search-icon"></i>
  <%= f.search_field :q, placeholder: text %>
  <% if button? %>
    <%= f.submit text %>
  <% end %>
<% end %>
```

### Organizing Component Source Files

To keep things tidy, we organize component files in a structured way. They are in the `app/views/components` folder. This includes both the Ruby class and the HTML template.

### Testing View Components

View components can be tested in isolation, improving the testability of our codebase.

```ruby
require "rails_helper"

RSpec.describe SearchBoxComponent, type: :component do
  let(:action) { '/search' }
  let(:text) { 'Search' }
  let(:button) { false }

  it "renders component" do
    render_inline(described_class.new(action:, text:, button:))

    expect(page).to have_css "input[type='submit']", text: "Search"
  end
end
```

### Dry Initializers for params

We can use the `dry-initializer` gem to declare parameters. It reduces repetition and helps readability. Additionally, View Component integrates seamlessly with Rails' internationalization system, allowing for localized components.

Let's create an Application View Component. It will be the main class for easy setup.

```ruby
class ApplicationViewComponent < ViewComponent::Base
 extend Dry::Initializer[undefined: false]
end
```

and update our component:

```ruby
class SearchBoxComponent < ApplicationViewComponent
 option :action
 option :text
 option :button, default: proc { false }

 private def button? = button
end
```

## TL;DR:
In Rails UI development, templates like HTML or Haml are crucial. But, they can be hard due to missing parameters and hidden dependencies.

Techniques like strict locals and linting help but aren't foolproof.

Breaking views into components, like React or Vue, is a solution. We can make Ruby classes for components. We will split logic from presentation and organize files in `app/views/components`.

Testing is improved with View Component helpers.

Using gems like `dry-initializer` improves readability. They also work with Rails' i18n system to aid localization.

> Learn more about this topic from book: [Layered Design for Ruby on Rails Applications](https://www.packtpub.com/product/layered-design-for-ruby-on-rails-applications/9781801813785)
{: .prompt-info }


