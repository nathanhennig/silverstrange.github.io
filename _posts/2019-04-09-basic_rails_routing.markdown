---
layout: post
title:  "Basic Routing in Rails"
author: "Nathan Hennig"
date:   2019-04-10 22:00:00 -0700
tags:   ruby rails routing
---

Code version used while writing this article:
  * **Rails Version:**  5.2.2  
  * **Ruby Version:**   2.6.0-rc1

Everything you need to know to get started with routing. If you are looking for more advanced routing information, skip to the end of this article for further reading links.

This article assumes that you have installed Ruby on Rails. If you have not, please visit the [Getting Started with Rails](https://guides.rubyonrails.org/getting_started.html) guide.

## What is routing?

Rails uses routes to sort incoming HTTP requests and direct those requests to specific endpoints. For basic routing, those endpoints will always be controller#actions. The Rails router can also generate paths and URLs so you can avoid hardcoding them in your views.



## Where are my routes?

You can see what routes currently exist by typing `rake routes` while in your rails project folder. Routing configuration is done in config/routes.rb. 

---

## Common Routing 

Let's start with a new Rails project.  
```
rails new routes
cd routes
```  

Take a look at what, if any, default routes exist.  
`rake routes`  

In Rails 5.2.2 I see the following five default routes related to Rails ActiveStorage. (ActiveStorage is intended to facilitate uploading files to the cloud and is outside the scope of this article.)

<div class="table-container" markdown="block">

Prefix                    | Verb | URI Pattern                                                 | Controller#Action
-------------------------:|:-----|:------------------------------------------------------------|:-----------------
rails_service_blob        | GET  | /rails/active_storage/blobs/:signed_id/*filename(.:format)  | active_storage/blobs#show
rails_blob_representation | GET  | /rails/active_storage/representations/:signed_blob_id/:variation_key/*filename(.:format) | active_storage/representations#show
rails_disk_service        | GET  | /rails/active_storage/disk/:encoded_key/*filename(.:format) | active_storage/disk#show
update_rails_disk_service | PUT  | /rails/active_storage/disk/:encoded_token(.:format)         | active_storage/disk#update
rails_direct_uploads      | POST | /rails/active_storage/direct_uploads(.:format)              | active_storage/direct_uploads#create

</div>

I left ActiveStorage on so I wouldn't get an empty result here, but you can use the `--skip-active-storage` switch when creating your project to disable ActiveStorage.

**Prefix** refers to the prefix for the path and URL helper names. With a prefix of `rails_service_blob` I can use `rails_service_blob_path` and `rails_service_blob_url`. Path helpers create site relative links that you'll use internally and URL helpers create absolute links that you can provided for external use (such as sending a link in an email notification).

**Verb** refers to HTTP verbs (get, post, patch, put, and delete). Depending on the HTTP verb used, the same URL may be directed to a different action.

**URI Pattern** refers to the pattern that a route will match (URI stands for Uniform Resource Identifier). These patterns have static segements like `/rails/active_storage/blobs/` that are hardcoded and dynamic segments like `:signed_id/*filename` that can hold a value. For example, `/rails/active_storage/blobs/345234/test_name_1234.txt` will be directed to the active_storage/blobs controller's show action and the params array will contain `params[:signed_id] = 345234 and params[:filename] = 'test_name_1234' params[:format] = 'txt'`.

**Controller#Action** identifies which action in which controller will handle the request. Actions are just methods/functions inside the controller class.

Use of parentheses indicates an optional parameter. The splat/glob operator `*` is used for [wildcard segments](https://guides.rubyonrails.org/routing.html#route-globbing-and-wildcard-segments) and is outside the scope of this article.

---

The most common routing directive is `resources`. If we generate a new resource:  
`rails generate resource Article`  

among other actions you'll see:  
> invoke  resource_route  
> &nbsp;&nbsp;route    resources :articles

Opening `config/routes.rb` will reveal a new `resources :articles` entry. Running the `rake routes` command will show eight new routes:  
(Note that the empty prefix entries mean 'same as above'. The verb changes but the prefix remains the same.)

<div class="table-container" markdown="block">

Prefix                    | Verb   | URI Pattern                  | Controller#Action
-------------------------:|:-------|:-----------------------------|:-----------------
articles                  | GET    | /articles(.:format)          | articles#index
|                           POST   | /articles(.:format)          | articles#create
              new_article | GET    | /articles/new(.:format)      | articles#new
             edit_article | GET    | /articles/:id/edit(.:format) | articles#edit
                  article | GET    | /articles/:id(.:format)      | articles#show
|                           PATCH  | /articles/:id(.:format)      | articles#update
|                           PUT    | /articles/:id(.:format)      | articles#update
|                           DELETE | /articles/:id(.:format)      | articles#destroy

</div>
<details><summary>Default Controller Actions Explanation</summary>

* index: show a list of resources/objects/models
* create: save a new object
* new: get a form for creating a new object
* edit: get a form for updating an object
* show: present a single object
* update: save an edited object
* delete: delete an object

This information is technically outside the scope of my article, but I feel that it is too often assumed that new users will understand the intent of these default actions.
</details>

The next most common routing directive is `resource`. This is used for singular resources that don't require an id parameter. For example, you may want `/profile` to always refer the currently logged in user, so you don't need or want an id provided in the URL.

Try editing `config/routes.rb` and changing `resources articles` to `resource article`, then run the `rake routes` command and examine the changes.

<details><summary>Changes</summary>

* The index route is removed since you can't have a list of a singular resource.
* The `:id` dynamic segment is removed from all the routes.
</details>

Individual routes can be created in the format of:  
`get '/photos/:pic_id', to: 'photos#show', as: 'photo_show'`

<div class="table-container" markdown="block">

Prefix                    | Verb   | URI Pattern                  | Controller#Action
-------------------------:|:-------|:-----------------------------|:-----------------
photo_show                | GET    | /photos/:pic_id(.:format)    | photos#show

</div>

The `:to` parameter sets the controller#action and is required. The `:as` parameter sets the prefix and is optional.

### Nesting

A common case is to have resources that are related to each other. For example, articles have many comments and a comment belongs to an article. In such a case you'll want to nest the resources to show the relationship.

```ruby
resources :articles do
  resources :comments
end
```

### Only and Except

Both `resources` and `resource` will accept the parameters `:only` and `:except` with an array. `:only` will limit the directive to only creating routes for the selected actions, while `:except` creates routes for the non-selected actions.


<div class="table-container" markdown="block">

`resources :articles, only: [:index, :create, :show]`

Prefix                    | Verb   | URI Pattern                  | Controller#Action
-------------------------:|:-------|:-----------------------------|:-----------------
articles                  | GET    | /articles(.:format)          | articles#index
|                           POST   | /articles(.:format)          | articles#create
article                   | GET    | /articles/:id(.:format)      | articles#show

`resources :articles, except: [:index, :create, :show]`

Prefix                    | Verb   | URI Pattern                  | Controller#Action
-------------------------:|:-------|:-----------------------------|:-----------------
new_article               | GET    | /articles/new(.:format)      | articles#new
edit_article              | GET    | /articles/:id/edit(.:format) | articles#edit
article                   | PATCH  | /articles/:id(.:format)      | articles#update
|                         | PUT    | /articles/:id(.:format)      | articles#update
|                         | DELETE | /articles/:id(.:format)      | articles#destroy

</div>

## Gotchas

When an incoming request is received, the Rails router finds the first matching route and follows it. If you make a change, but your url is still going to the same place, make sure you haven't created and overlapping route patterns

I'll add more here as I think of them. Please comment below if you have any you'd like me to add.

## Further Reading

* Official [Ruby on Rails Routing Guide](https://guides.rubyonrails.org/routing.html) - This is a great resource and you can find pretty much anything you want to know about Rails routing here.
* GitHub - the Rails source code has excellent comments although the material is mostly the same as the Rails Routing guide.
	* [routing.rb](https://github.com/rails/rails/blob/master/actionpack/lib/action_dispatch/routing.rb)
	* [mapper.rb](https://github.com/rails/rails/blob/master/actionpack/lib/action_dispatch/routing/mapper.rb)
* [Scope vs Namespace](https://devblast.com/b/rails-5-routes-scope-vs-namespace) - Scope and namespace are very similar routing methods. This article by Thibault gives a solid explanation of the differences between the two.