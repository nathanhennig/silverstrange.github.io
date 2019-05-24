---
layout: post
title:  "Presenters in Rails"
author: "Nathan Hennig"
date:   2019-05-23 15:30:00 -0700
tags:   ruby rails presenter mvc
---

Code version used while writing this article:
  * **Rails Version:**  5.2.2  
  * **Ruby Version:**   2.6.0-rc1

## What are Presenters?

Rails is based on the [Model-View-Controller](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller){:target="_blank"} (MVC) architectural pattern. Models store and manage data, views present the model to the user, and controllers respond to user input and interact with model data.

The presenter concept comes from the [Model-View-Presenter](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93presenter){:target="_blank"} (MVP) architectural pattern which is derived from the MVC pattern. Presenters are essentially controllers that have specifically be assigned responsibility for display logic.

With any architectural pattern, there will always be debates on where specific responsibilities belong. In the Rails applications I've worked on, I've mostly seen a mix of people assigning display logic to the view, to the controller, or (worst of all) both. I personally prefer to put display logic in the views and view helpers, but this can lead bloated views that are hard to read, understand and alter.

To combat this problem, we can use presenters alongside our regular MVC architecture and move all the display logic from our views and helpers to our presenters.

## How to Use Presenters

Ryan Bates' excellent RailsCast [Presenters from Scratch](http://railscasts.com/episodes/287-presenters-from-scratch){:target="_blank"} gives a detailed explanation of how to use presenters in Rails. Most of the following content will be a simplified rehash of his video, so I highly recommend visting the source for more details.

Presenters go in the app/presenters folder, so create it if it doesn't exist. Presenters should be named in the format `model_presenter.rb` with a class name of `ModelPresenter`.

### BasePresenter
Next we'll need a BasePresenter class to abstract out common functionality that we'll reuse in all of our presenters.

```ruby
# app/presenters/base_presenter.rb
class BasePresenter
  def initialize(object, template)
    @object = object
    @template = template
  end

private

  def self.presents(name)
    define_method(name) do
      @object
    end
  end

  def h
    @template
  end
 
  def method_missing(*args, &block)
    @template.send(*args, &block)
  end
end
```

The `h` method is a pattern from Draper. The `method_missing` method sets the template (view object) as the fallback for any missing methods in the presenter. This allows use to use ActiveView methods with `h.image_tag` or just `image_tag`.

### Presenter Class

```ruby
# app/presenters/user_presenter.rb
class UserPresenter < BasePresenter
  presents :user
  delegate :username, to: :user

  def avatar
    site_link image_tag("avatars/#{avatar_name}", class: "avatar")
  end

  def linked_name
    site_link(user.full_name.present? ? user.full_name : user.username)
  end

private

  def site_link(content)
    h.link_to_if(user.url.present?, content, user.url)
  end

  def avatar_name
    if user.avatar_image_name.present?
      user.avatar_image_name
    else
      "default.png"
    end
  end
end
```
The `presents` class method functions as an alias for `@object` so we can write `user.full_name` instead of `@object.full_name`, for example.

### Helper Methods
Now we need a couple helper methods to let us access the presenter.

```ruby
# app/helpers/application_helper.rb
module ApplicationHelper
  def present(object, klass = nil)
    klass ||= "#{object.class}Presenter".constantize
    presenter = klass.new(object, self)
    yield presenter if block_given?
    presenter
  end
end
```

The `present` method in ApplicationHelper lets us access the presenter from our views. But we might also need to access it in the controller, so we need to define another `present` method in the ApplicationController.

```ruby
# app/controllers/application_controller.rb
def present(object, klass = nil)
  klass ||= "#{object.class}Presenter".constantize
  klass.new(object, view_context)
end
```

Now we can call `present` in any controller which inherits from ApplicationController (usually all of them).

### View

```rails
<%# app/views/users/show.html.erb %>
<% present @user do |user_presenter| %>
  <div id="profile">
    <%= user_presenter.avatar %>
    <h1><%= user_presenter.linked_name %></h1>
    <dl>
      <dt>Username:</dt>
      <dd><%= user_presenter.username %></dd>
    </dl>
  </div>
<% end %>
```

By using the presenter we move out all the logic from view. This makes the view easy to read. If we need to restructure the view, there's no need to worry about breaking logic flows. The code is also DRYier and easy to re-use elsewhere if needed.

### Testing with Rspec
Create the `spec/presenters` folder. To test the presenters, we'll need to include `ActionView::TestCase::Behavior` which gives us the `view` object. Rather than including this module in each presenter, add the following to your spec_helper.

```ruby
# /spec/spec_helper.rb
RSpec.configure do |config|
  config.include ActionView::TestCase::Behavior, example_group: {file_path: %r{spec/presenters}}
  # Rest of block omitted.
end
```

Create a `user_presenter_spec`.

```ruby
# /spec/presenters/user_presenter_spec.rb
require 'spec_helper' # or 'rails_helper' depending on setup

describe UserPresenter do
  it "says when none given" do
    presenter = UserPresenter.new(User.new, view)
    presenter.website.should include("None given")
  end
end
```

Note that the view object can not access helper methods in the controller, such as `current_user` so they'll have to be stubbed out if used by the presenter.

## Further Reading/References

* Railscasts 287 - [Presenters From Scratch](http://railscasts.com/episodes/287-presenters-from-scratch){:target="_blank"}
* Wikipedia - [Model-View-Controller](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller){:target="_blank"}
* Wikipedia - [Model-View-Presenter](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93presenter){:target="_blank"}
