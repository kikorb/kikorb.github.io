---
layout: post
title:  "Mounting rails cells in rails routes"
date:   2015-05-19 19:59:46
categories: rails cells routes
---
I am all in with rails cells, the idea is simple and clean. Modularizing and isolating are always good calls, specially in large projects. But what if you have a cell that renders a part of your application and you want to update it using ajax.

```ruby
class UserInfoCell < Cell::ViewModel
  def show(options)
    @user = User.where(id: options[:id])
    render
  end
end
```

Imaging a cell that renders the user info of your website with the remaining credit. You might want to pool or use sockets to be get the latest info for the user.

In order to do that you will need a separate controller that gives you either the html that needs to replace the current one or the json data for you to do it in the client side.

With cells you can also mount the cell in the routes.rb file like the documentation specifies and be able to call that cell action directly.

> Cells 3.8 got rid of the ActionController dependency. This essentially means you can mount Cells to routes or use them like a Rack middleware. All you need to do is derive from Cell::Base.

```ruby
class PostCell < Cell::Base
  ...
end
```

In your routes.rb file, mount the cell like a Rack app.
```ruby
match "/posts" => proc { |env|
  [ 200, {}, [ Cell::Base.render_cell_for(:post, :show) ]]
}
```
But what if you want to use Cell::Rails instead of Cell::Base and still get the cell to be mountable in the routes.

Well you can:

```ruby
match "/posts" => proc { |env|
  controller = ActionController::Base.new
  controller.request = ActionDispatch::Request.new(env)

  [ 200, {}, [ Cell::Rack.render_cell_for(:post, :show, controller, , controller.request.query_parameters) ]]
}
```

Cell::Rails inherits from Cell::Rack and Cell::Rack inherits from Cell::Base. The main difference between the Base and the Rack class is that the class is expecting a controller to delegate things like the session, the request object, etc, things that you might need to use devise in you cells or any other Rails helper.

This is performs almost as fast as having a metal, places all the necessary logic inside the cell avoiding us from creating an extra controller in the application to deal with the ajax calls to update the cell.

Simple clean and elegant, and without loosing the Cell::Rails in the way.
