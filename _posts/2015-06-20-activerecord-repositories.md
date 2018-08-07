---
layout: post
title: "ActiveRecord Repositories?"
description: "There is no one who loves pain itself, who seeks after it and wants to have it, simply because it is pain..."
comments: true
keywords: "dummy content, lorem ipsum"
---

For the last couple of weeks I have been refactoring an app at work, and the idea of using a repository object to constrain the access to the db has been on my mind the whole time.
We're currently boarding the hexagonal architecture wagon, and after watching [Architecture: The Lost Years]() and [Decoupling from Rails]() they gave us a little insight on which way to go. This is a not-so-green-field app, and we are using Rails, of course, with ActiveRecord. This means a lot of work with measly gains., But sometimes you can't fight the river you are swimming on so we decided to embrace AR.

And then, when working with this simple interactor object, I noticed this repeated pattern of using the ever so common:

```ruby
model = Model.find(13)
if model.update(model_params)
  # things went well
else\t
  # something went wrong
end
```

Because of `.create` I wondered \"wouldn't it be nice if there was an `.update` or a `.destroy`?\" and in fact there are. The catch is that these methods always return an instance whether or not they were saved to the db. So now the code looks like this:

```
model = Model.create(model_params)
if model.persisted?
  # things went well
else
  # something went wrong
end
```

Which I think is sort of nice, I'm using ActiveRecord::Base class methods to define the API of my \"repository\" and only using the model instance as an \"entity\" (Yes, sure, I still have persistence logic in the entity. And  in fact, I'm using it, but that can be a little to much to ask of AR).

Before you close the post, yes i know these methods actually do the same thing I was doing before, but it serves as a \"boundary\" between the two, and although it's not perfect, it leaves us with a clear protocol (or port, in [Cockburn's terms](alistair.cockburn.us/Hexagonal+architecture)) which would allow us to switch away from Active::Record easily.


## There are  some problems with this though:

#### We don't have a method to save an entity to the db:
```
model.active = false
Model.save(model)
```

A `.save` method would be a nice addition to these methods, otherwise we would have to use the `.update`, with all the changes made to the object; or defining our own class method to save these differences to the db.

Because our models serve two purposes i wouldn't want to inherit from any new class, I would do something along the concerns lines:
```
module Repository
  def save(entity)
    entity.save
  end
end

class Order < ActiveRecord::Base
  extend Repository
end
```

#### Applying authorization policies can get in the way.

In this project, I'm working with [Pundit](https://github.com/elabs/pundit) and normally when editing or destroying a resource we first fetch the resource, ask our policies if we are allowed to update/destroy that resource and then, and only then, update/destroy said resource.

```
order = Order.find(order_id)
policy(order, :update?)
order.update(order_params)
```

But, because now you are doing the fetching and the updating in the same command you'd have to customize your policy to use the resource's id instead of the object and make the minimum query possible. This probably means that your queries aren't going to be pretty:
```
class PagePolicy < ApplicationPolicy
  def show?
    manager? || user_can_access_pages?
  end

  def user_can_access_pages?
    user.clients.includes(papers: :pages).pluck('pages.id').include?(record_id)
  end
end
```
```
policy(order_id, :update?)
Order.update(order_id, order_params)
```

#### Eager loading associations

There are cases where you want to fetch the object and its  associations in a single query. For that ActiveRecord uses the `.includes`, `.preload`, `.eager_load` methods which we cannot use on the instance.

Aesthetically speaking `Newspaper.includes(:pages).update(newspaper_id, newspaper_params)` looks and reads horrible.

And this breakes the whole boundary idea we spoke about earlier.

###So,
Let me know, on the comments section or on twitter,  what you think about what you just read. This is just an idea that I've been fiddling about on paper, no real attempt has been made to achieve this **YET**.