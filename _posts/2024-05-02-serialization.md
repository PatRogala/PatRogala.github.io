---
title: How to Master JSON Serialization in Rails
date: 2024-05-02 16:54:00 +0000
categories: [Ruby on Rails, Layered Design for Ruby on Rails Applications]
tags: [ruby, rails, web, layered design, models, active record, json, serializers]
---

In the digital age, data is crucial.

JSON serialization in Ruby on Rails is critical. It transforms complex data into easy-to-use information. You may be building a light mobile app or a full web solution. In either case, mastering JSON serialization is vital.

Today, we will learn basic and advanced techniques to streamline data exchange in Rails. We'll cover `#as_json` settings and advanced uses of Jbuilder and Alba.

## Turning Models into JSON in Rails

Rails provides easy methods to create JSON responses. Here’s a simple way to do it:

### Using `#as_json`

You can quickly generate JSON from your Rails controllers. Here's how you can send a JSON response:

```ruby
class PostsController < ApplicationController
  def show
    post = Post.find(params[:id])
    render json: post
  end
end
```

This command converts the model into JSON. But, it includes all model attributes, which you might not want to expose, like user passwords.

To control which attributes to include, you can change the `#as_json` method like this:

```ruby
class Post < ApplicationRecord
  belongs_to :user
  def as_json(options={})
    super({
      only: [:id, :title, :draft],
      include: { user: { only: [:id, :name] } }
    }.merge(options))
  end
end
```

With this setup, you can specify which fields to include and even add related data:

```ruby
# Example output
{ "id": 1, "title": "Serializers", "draft": true, "user": { "id": 1,"name": "J Doe" }}
```

### Using Jbuilder
For complex JSON structures, Rails uses Jbuilder, a template engine. This allows you to create JSON templates from your model logic. This way, you don't need to change your models to change JSON outputs.

> Learn more about [jbuilder](https://github.com/rails/jbuilder)
{: .prompt-info }

```ruby
# app/views/posts/show.json.jbuilder

json.id @post.id
json.title @post.title
json.draft @post.draft

json.user do
  json.id @post.user.id
  json.name @post.user.name
end
```

## Serializers in Ruby

In Ruby, serializers help turn objects into JSON for API responses. They act like presenters, only sending what the client needs, not the original objects.

### Simple Ruby Serializers

You can make a Ruby class act as a serializer by defining an `#as_json` method. Here’s an easy way to write a serializer for a user:

```ruby
class UserSerializer
  def initialize(user)
    @user = user
  end

  def as_json
    {id: @user.id, name: @user.first_name}
  end
end
```

### Using Alba for Advanced Serializing

Alba is a Ruby library that makes serializing objects straightforward and efficient. It lets you define what parts of the object to turn into JSON. Here’s an example of how to use Alba for a post serializer:

```ruby
class PostSerializer
  include Alba::Resource

  root_key :post

  attributes :id, :title
  attribute :is_draft, &:draft?

  one :user, resource: UserSerializer
end
```

This setup includes user data with the post. It uses predefined settings to simplify coding.

In your controller, you can use these serializers:

```ruby
class PostsController < ApplicationController
  def index
    render json: serialize(Post.all)
  end
end
```

> Define helper method `serialize` in **ApplicationController** to provide easy access to serializers in every controller
{: .prompt-tip }

The serialize method uses the serializer based on the object's class. This method makes it easy to manage JSON structures and keeps the code clean and organized.

## TL;DR: JSON Serialization Techniques in Ruby on Rails

Rails makes JSON serialization easy. It does this via `#as_json` and **Jbuilder**. They allow selective attribute inclusion and complex structure creation.

For deeper control, Ruby's serializers like **Alba** customize JSON outputs. Learn to improve your Rails applications, by turning models into JSON. This ensures secure and flexible data handling.

> Learn more about this topic from book: [Layered Design for Ruby on Rails Applications](https://www.packtpub.com/product/layered-design-for-ruby-on-rails-applications/9781801813785)
{: .prompt-info }


