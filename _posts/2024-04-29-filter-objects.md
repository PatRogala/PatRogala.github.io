---
title: Filter Objects in Rails - How to Manage Filtering
date: 2024-04-29 14:29:55 +0000
categories: [Ruby on Rails, Layered Design for Ruby on Rails Applications]
tags: [ruby, rails, web, layered design, models, active record, filter object]
---

Ruby on Rails development is always changing. Managing complexity well is a key.

Rails developers often find themselves wrestling with bloated controllers. This is especially true when filtering data based on user input.

The old way clutters your controllers. It also makes them error-prone and hard to maintain.

But what if there was a cleaner way to filter? It could greatly simplify your controllers and make your app's architecture more elegant.

**Filter Objects**. They are a big shift in Rails. They promise to streamline your code and focus you on what truly matters.

Let’s explore how to use them in your Rails apps. It makes your controllers sleek, efficient, and very easy to manage.

## Simplifying Controller Filtering in Rails

In Rails, controllers often filter based on user input. But, this can make them complex and hard to manage.

### Example: Simplified Project Filtering

Imagine a controller that filters project listings based on project status or date:

```ruby
class ProjectsController < ApplicationController
  def index
    @projects = filter_projects(params)
    render json: @projects
  end

  private

  def filter_projects(params)
    projects = Project.all
    projects = projects.where(status: params[:status]) if params[:status].present?
    projects = projects.where('started_at > ?', Time.current) if params[:time_filter] == "future"
    projects = projects.where('name LIKE ?', "%#{params[:search]}%") if params[:search].present?
    projects
  end
end
```

This method starts with all projects. Then, it applies filters based on the given parameters. This setup keeps the index action clean. It moves the detailed filtering logic into a separate method.

### Why It’s Better

This way reduces complexity in the index method. It makes the filtering process easier to manage and test. Each condition is clearly separated. This makes modifications and testing easier.

## Model Filtering with Scopes in Rails

Moving filtering logic from controllers to models can streamline your Rails application. Here’s how you can use scopes to tidy up the filtering process:

### Simplified Model Scopes Example

For a project model, we can define specific scopes for each type of filter:

```ruby
class Project < ApplicationRecord
  scope :by_status, -> (status) { where(status: status) if status.present? }
  scope :future_projects, -> { where("started_at > ?", Time.current) }
  scope :by_name, -> (name) { where("name LIKE ?", "%#{name}%") if name.present? }
end

```

The scopes handle filtering by project status. They also filter by future start dates and project name. Each scope only applies if you provide the relevant parameters. This keeps queries efficient and relevant.

### Cleaner Controller Implementation

The model handles the detailed filtering logic. This makes the controller simpler:

```ruby
class ProjectsController < ApplicationController
  def index
    @projects = Project.all.by_status(params[:status])
                         .future_projects
                         .by_name(params[:name])

    render json: @projects
  end
end
```

This approach keeps the controller lean. It does this by shifting filtering to the model. It uses chainable scopes. They make the code easier to read and maintain.

## Introducing Filter Objects in Rails

In Rails, handling complex filtering in the controller or model can lead to bloated code. It can also make the code hard to test. To simplify, we can introduce a concept called a filter object. This object does all the filtering. It lets the controller stay clean and focused on requests.

### Example: Using a Filter Object

Here's how we can refactor a Rails controller to use a filter object for handling project filters:

```ruby
class ProjectsController < ApplicationController
  def index
    @projects = ProjectFilter.apply(Project.all, params)
    render json: @projects
  end
end
```

This controller now uses `ProjectFilter.apply` to handle filtering based on parameters. This change significantly simplifies the `index` method.

### Creating a Basic Filter Object

The filter object encapsulates all parameter parsing, sanitizing, and filtering:

```ruby
class ProjectFilter
  def self.apply(scope, params)
    scope = scope.where(status: params[:status]) if params[:status]
    scope = scope.where("started_at > ?", Time.current) if params[:time_filter] == "future"
    scope = scope.where("name LIKE ?", "%#{params[:query]}%") if params[:query]
    scope
  end
end
```

This class method is on `ProjectFilter`. It takes a base scope, like `Project.all`, and adds filters based on the provided `params`.

### Benefits of Filter Objects

Using a filter object makes the controller's code easier to read and test. It delegates the job of filtering data to a separate object. **This object is dedicated**. It can be tested and reused in different parts of the application. This separation of concerns follows the principles of clean architecture. It improves the maintainability and scalability of your application.

## Differences between Filter, Form, and Query Objects in Rails

> Check out post about Form Objects [HERE](https://patrogala.github.io/posts/form-objects/)
{: .prompt-info }

> Check out post about Query Objects [HERE](https://patrogala.github.io/posts/Layered/)
{: .prompt-info }

In Rails, it's helpful to distinguish between object types used for handling data. There are filter, form, and query objects. Each serves unique purposes:

### Filter Objects
- **Purpose**: Filter objects process user inputs. They refine data queries, like filtering a list of projects by status.
- **Usage**: They take user input directly and adjust database queries accordingly. Filter objects can use query objects. But, they are more connected to user inputs.

### Form Objects
- **Purpose**: Form objects handle form submissions. They manage data validation and actions. These actions include creating or updating records.
- **Behavior**: They show if actions succeed or fail. For example, they show form validation errors. They are directly involved with user interactions.

### Query Objects
- **Purpose**: Query objects are purely for building and executing database queries.
- **Scope**: They work at a lower level, dealing only with domain objects and not directly with user inputs.

## Conclusion

Filter object keeps the controllers light and focused only on handling requests. It delegates the complex filtering logic to dedicated filter objects.

This strategy has two benefits. It simplifies the codebase by removing complex if-else logic from controllers. It also improves the application’s maintainability and scalability.

Filter objects centralize the filtering logic. They improve code readability and testability.

This makes the code easier to modify and extend as requirements change.

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->

> Learn more about this topic from book: [Layered Design for Ruby on Rails Applications](https://www.packtpub.com/product/layered-design-for-ruby-on-rails-applications/9781801813785)
{: .prompt-info }

<!-- markdownlint-restore -->
