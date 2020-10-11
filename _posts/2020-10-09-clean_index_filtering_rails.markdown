---
layout: post
comments: true
title:  "Clean index searching/filtering in Rails"
date:   2020-10-09 09:12:15
category: Technology
tags: [Ruby, Rails, API, Search]
image: /assets/images/GAP_lake.jpg
image2: /assets/images/GAP_lake_mobile.jpg
---

Just about every web application I've worked on has required some form of searching. Its pretty much a given that users will want to find _things_ in your application by providing some refining criteria.

A fairly common approach to searching/filtering on users might look something like this.

<script src="https://gist.github.com/johnmfarrell1/bef96af4de1846fb58ad45cec973f937.js"></script>

Ugh.... the problem here is every time you want to add a new search field, you end up bloating the index action of your controller with more of the same, repetitive code.
You could move all this code in to something like a search service (e.g. `UserSearch.call(search_params)`), but you'd also end up with the same boilerplate code that would grow and grow.

Clearly there is a pattern here that could be refined into a reusable concern so we can do this in a less repetitive way. 
Justin Weiss's great article [Search and Filter Rails Models Without Bloating Your Controller](https://www.justinweiss.com/articles/search-and-filter-rails-models-without-bloating-your-controller/) describes just how you can achieve this by using scopes and taking advantage of `ActiveRecord`'s method chaining so that very same code above would end up looking like as follows.

<script src="https://gist.github.com/johnmfarrell1/6e37c3fc1d1d197febabb7be29032a1c.js"></script>

Much cleaner! I'd highly recommend reading that article to get a good idea of how its achieved.
In any case, here's what Justin's concern looks like (you simply include it in the model you want to add filtering on):

<script src="https://gist.github.com/johnmfarrell1/2536d867cfcd9de72726732d19a123bf.js"></script>

I've used a modified version of this concern in numerous applications and its really helped how cleanly and simply more search criteria can be added without bloating the controller action.
So lets take a look at some of the main modifications to the concern we've made.

## 1. Automatic whitelisting of filtering scopes.
While using this concern in practice (while working on an advanced search page) we had _many_ different filtering criteria possible on the index pages we were adding searching to.
The problem we found was that it became tedious to have to keep track of what filtering scopes we wanted to whitelist and we had to keep incrementally changing `filtering_params` in the controller.
At the start, this wasnt so much of a problem but as the list grew, it became a bit hard to manage. It could also easily fall out of sync with the scopes we had defined and the ones we had whitelisted for the index.

So, in order to address this, we added a new specific method for adding these scopes via `filter_scope` and an associated array which we would add the scope name to on each class its used in.

The modification looks as follows:

<script src="https://gist.github.com/johnmfarrell1/12f45847ade1463f64e38e063651bc64.js"></script>

This essentially wraps the `scope` call so that we can now keep track of all `filter_scopes` we define.
So, its simply used as you would a normal scope.
```ruby
filter_scope :country, ->(country) { where(country: country) }
```
Now, we can get a list of all `filter_scopes` via `User.filter_scopes`. Super useful for using in our controller to permit the params.
It also has the added benefit of explicitness in the model so you can clearly keep scopes intended for use in the filtering separated from any other scopes.


## 2. Removing 'filter_by_' prefix
As a result of having a mechanism for automatic whitelisting of filtering scopes, it meant that we could remove the hardcoded prefix requirement for having to use `filter_by_<attribute_name>`.


## 3. Handling arrays with blank items
We came across some scenarios where arrays of elements would end up causing queries with blank strings if the array values contained blanks.
In Justins original `filter` method, he had check for whether a value was blank or not.
We simply applied the same logic for arrays by adding the following line so that blank elements would be rejected: 
```ruby
filter_value = filter_value.reject(&:blank?) if filter_value.is_a?(Array)
```

## The complete concern and example usage

So, heres what the modified concern looks like now...

<script src="https://gist.github.com/johnmfarrell1/189d05b8095811cb5f1e56453852c457.js"></script>

Example usage in a `User` model.

<script src="https://gist.github.com/johnmfarrell1/61ea642241de98fce29f7e511f450040.js"></script>

Because our concern is keeping track of scopes we intent on using for filtering, it means we can get a list of them by doing the following...
```ruby
User.filter_scopes
# => [:first_name, :last_name, :country]
```

This is super useful because it is our whitelist and can then be used in our controller to permit the params.
Here's what the controller looks like.

<script src="https://gist.github.com/johnmfarrell1/46e35f135ed538ba5d08aaf6dd74d99c.js"></script>

What's really great about this solution that when you want to add more criteria to filter by, you no longer need to make changes to the controller.
It also addresses Justins concern regarding how easy it is to forget param whitelisting - you no longer need to maintain a list, it is automatic.
So, adding a new filtering criteria is as easy as this...

```ruby
filter_scope :email, ->(email) { where(email: email) }
```

And conveniently, our `filter_scopes` user method also keeps track so it is automatically permitted and our controller will work without any changes...
```ruby
User.filter_scopes
# => [:first_name, :last_name, :country, :email]
```

Pretty neat! 

For a working example of this in action, you can take a look at the following repo:
[https://github.com/johnmfarrell1/rails_index_filtering](https://github.com/johnmfarrell1/rails_index_filtering)

In the next post, I'll describe an approach to the Query Object pattern and how it can be easily wired up to also use this concern in a nice clean way so that scopes and scope logic can be moved out of the model to avoid the model getting too fat if necessary.
