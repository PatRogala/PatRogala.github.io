---
title: Simplifying Web Application Security
date: 2024-05-04 09:26:00 +0000
categories: [Ruby on Rails, Layered Design for Ruby on Rails Applications]
tags: [ruby, rails, web, layered design, models, authorization, authentication, web security]
---

## Understanding Authentication and Authorization

Authentication is about verifying who's making a request. It's like checking someone's ID to confirm their identity. Once verified, we issue a token for future requests—this is authentication. On the other hand, authorization determines what actions a user can perform. It checks if the identified user has the right permissions to do something.

### Example in Action

Let's say we have a controller action for deleting a post. We use authentication to verify the user's identity and authorization to check if they're allowed to delete the post.

```ruby
class PostsController < ApplicationController
  before_action :authenticate! # authentication

  def destroy
    post = Post.find(params[:id])
    if post.user_id == current_user.id # authorization
      post.destroy
      redirect_to posts_path, notice: "Post has been deleted"
    else
      redirect_to posts_path, alert: "You are not allowed to delete this post"
    end
  end
end
```

## Securing Different Layers

![Desktop View](/posts/images/secure-layers-of-web-app.webp){: width="972" height="589" }
_Secure Layers of Web App_

Securing a web app involves protecting various layers. Beyond just authorization and authentication, we also need to consider system constraints and data validation.

### System Constraints
These are rules that apply regardless of the user. For instance, limiting the number of actions a user can perform per day or enforcing specific business rules.

### Data Validation
At the domain level, we ensure data consistency through validations. This ensures that the data meets certain criteria before being processed.


> Learn more about this topic from book: [Layered Design for Ruby on Rails Applications](https://www.packtpub.com/product/layered-design-for-ruby-on-rails-applications/9781801813785)
{: .prompt-info }


