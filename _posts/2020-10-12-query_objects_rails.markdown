---
layout: post
comments: true
title:  "Query Objects in Rails"
date:   2020-10-12 09:12:15
category: Technology
tags: [Ruby, Rails, Query, ActiveRecord]
image: /assets/images/singapore_merlion_park.jpg
image2: /assets/images/singapore_merlion_park_mobile.jpg
---

Query objects are not a particularly new idea, they've been around a while and in many languages.
Simply put, a query object is just a class used to encapsulate a piece of query logic.
Traditionally, in Rails, query logic typically lives in the relevant `ActiveRecord` model class as a scope or in some cases as a separate service class that performs a query and returns _something_ depending on the need (I say something as service objects tend to be quite general by nature).

### Building a Query object from an existing scope

Say we have an existing scope, which looks as follows:

<script src="https://gist.github.com/johnmfarrell1/fea159a70ae1f2509a75754743fc774a.js"></script>

Lets say we want to move this to a query object. First step, lets create a place to put our query objects.
I'd recommend a new folder under the `app` called `queries` so its nice and explicit whats in there.
The above scope refactored as a query object might look a little bit like as follows:

<script src="https://gist.github.com/johnmfarrell1/7e7658974d658ec8a63b74ec65304752.js"></script>

This _kind of_ works. I mean its encapsulated our query logic, but its not very flexible as we can mix it with other queries like we would typically do when chaining scopes.
We could make this a little bit more flexible by giving the query object some state so that it can be passed a relation object to work on (with a default so its optional).
We'll still keep the main invoking method name as `call` which is a de facto practice in Ruby.
It might look something like as follows:

<script src="https://gist.github.com/johnmfarrell1/7866c8a74edb66251f34c20dc5553955.js"></script>

Now, it's a little bit more flexible because we can pass in an existing collection and it will work off of that.

For example, something like this `UserPromotedPostsQuery.new(User.country('Ireland')).call`.
In other words, Users with promoted posts from Ireland. 

It would probably make sense to namespace this query also under the `User` namespace as you'd imagine our `queries` directory would get quite unmanageable as we add more to it.
In that case, the query object would be referenced via `User::PromotedPostsQuery` and live in `app/users/promoted_posts_query.rb`.

As we create more queries, you can imagine that theres going to be some copy and pasting each time, as the initializer is going to be nearly the same for each one.
The only aspect that will change is the subject, which can be implied by the name space in any case.

So, how would a `BaseQuery` class look that all query objects could inherit from.
It might look something like this..

<script src="https://gist.github.com/johnmfarrell1/5f3107ab470d3192c54d075c62b9eb3e.js"></script>

Now, our initializer defaults to the module parent, so once we properly namespace our queries under the `ActiveRecord` subject class, this will automatically work and we can override this if desired.
We simply need to implement the `call` method. The `call` in our base class is simply a template. 

So, now our `User::PromotedPostsQuery` would look as follows:

<script src="https://gist.github.com/johnmfarrell1/37b17681931b6d84a7ea3668ca8031d7.js"></script>

Nice and to the point but still flexible. We can still do stuff like `User::PromotedPostsQuery.new(User.country('Ireland')).call`.
We've also made a tiny refactoring to use `merge` so the `Post.promoted` logic lives under the `Post` where it should.

### Wiring our query object up to our model

If we wanted to chain our query objects together so we can build up useful queries, you can imagine it would get a little bit long winded.
For example 
```ruby
User::IsActive.new(User::PromotedPostsQuery.new(User.country('Ireland')).call).call
```
i.e. Active users with promoted posts from Ireland.

Ugh... I guess we dont have to do it all in one line, but there is a nicer way to hook these query objects to our model class.

When we look at the `ActiveRecord` definition of a named `scope`, we see something interesting.
```ruby
# File activerecord/lib/active_record/scoping/named.rb, line 170
def scope(name, body, &block)
  unless body.respond_to?(:call)
    raise ArgumentError, "The scope body needs to be callable."
  end
  # ...
end
```

This tells us that we can actually pass a body that responds to `call` and follows the rules of what scopes do (i.e. always return collections) and it'll all work.
So, conveniently, we can actually do the following...

<script src="https://gist.github.com/johnmfarrell1/ab967304b0daa65f7b048fd7e8772eb9.js"></script>

So, now we can do `User.active.with_promoted_posts.country('Ireland')`.
This is much more readable now.

We can make one small adjustment to our `BaseQuery` class so we don't need to always initialize it (as in Ruby, even classes are objects).

We can add the following:

```ruby
class << self
  delegate :call, to: :new
end
```

By doing this, it means wiring it up to our model is a little cleaner
```ruby
scope :with_promoted_posts, PromotedPostsQuery
```

Its also possible to write query objects that take some variables via the call method.
For example, we might have a query object that fetches users that have made comments of a particular status.
i.e. `User::CommentStatusQuery.call('pending')`
In other words, Users that have pending comments. For scenarios like this we have 2 options when wiring them up to the model,

Option 1, as above...
```ruby
scope :with_comment_status, CommentStatusQuery
```

The downside to this approach is that its not obvious that a parameter is required. 
In this case if we wanted to make it more explicit, we could re-writing it as follows:

```ruby
scope :with_comment_status, ->(status) { CommentStatusQuery.call(status) }
```

Both approaches work and I guess it depends on preferences which way is preferred.

### Merging with the filterable concern

In my [previous post](https://ifiwere.me/technology/2020/10/09/clean_index_filtering_rails.html), I ran over an approach to filtering on an index using a `Filterable` concern.
Conveniently, we can merge our query objects with our filterable concern as follows.

```ruby
filter_scope :with_comment_status, CommentStatusQuery
```

Now, `with_comment_status` is a filtering param we can automatically use in our controller index.
So, we'll now be able to go to `www.site.com/users?with_comment_status=pending` and it will invoke our query object.

## The complete BaseQuery with examples
So, heres what it all finally looks like...

<script src="https://gist.github.com/johnmfarrell1/31f42f14e3b2f02d5c24ae33f66e7eda.js"></script>
<script src="https://gist.github.com/johnmfarrell1/192ad43c29bd1c5e897082d04de584ae.js"></script>
<script src="https://gist.github.com/johnmfarrell1/7fb12b204192548cad403e441303a0f9.js"></script>
<script src="https://gist.github.com/johnmfarrell1/53d9a6c4e58d0e5eb55b4f38c240ff59.js"></script>

For a working example of this in action, you can take a look at the following repo:
[https://github.com/johnmfarrell1/rails_index_filtering](https://github.com/johnmfarrell1/rails_index_filtering)