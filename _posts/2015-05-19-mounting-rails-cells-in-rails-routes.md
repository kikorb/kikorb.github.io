---
layout: post
title:  "Mounting rails cells in rails routes"
date:   2015-05-19 19:59:46
categories: rails cells routes
summary: "I am all in with rails cells, the idea is simple and clean. Modularizing and isolating are always good calls, specially in large projects. But what if you have a cell that renders a part of your application and you want to update it using ajax..."
---
I am all in with rails cells, the idea is simple and clean. Modularizing and isolating are always good calls, specially in large projects. But what if you have a cell that renders a part of your application and you want to update it using ajax.

{% highlight ruby %}
class PostCell < Cell::Rails
  def show(options)
    @posts = Post.where(published: options[:published]).where(locale: session[:locale])
    render
  end
end
{% endhighlight %}

Imaging a cell that renders the posts of your website scoped by the locale. You might want to pool or use sockets to be get the latest posts every x seconds.

In order to do that you will need a separate controller that gives you either the html that needs to replace the current one or the json data for you to do it in the client side.

With cells you could mount the cell in the routes.rb file like the [documentation specifies](https://github.com/apotonick/cells#mountable-cells) and be able to call that cell action directly.

> Cells 3.8 got rid of the ActionController dependency. This essentially means you can mount Cells to routes or use them like a Rack middleware. All you need to do is derive from Cell::Base.

{% highlight ruby %}
class PostCell < Cell::Base
  ...
end
{% endhighlight %}

> In your routes.rb file, mount the cell like a Rack app.
{% highlight ruby %}
match "/posts" => proc { |env|
 [ 200, {}, [ Cell::Base.render_cell_for(:post, :show) ]]
}
{% endhighlight %}

But what if you want to use Cell::Rails instead of Cell::Base and still get the cell to be mountable in the routes.

Well you can:

{% highlight ruby %}
match "/posts" => proc { |env|
  controller = ActionController::Base.new
  controller.request = ActionDispatch::Request.new(env)

  [ 200, {}, [ Cell::Rack.render_cell_for(:post, :show, controller, controller.request.query_parameters) ]]
}
{% endhighlight %}

The trick is to provide a controller with the right environment/request.

Cell::Rails inherits from Cell::Rack and Cell::Rack inherits from Cell::Base. The main difference between the Base and the Rack class is that the Rack flavour is expecting a controller to delegate things like the session, the request object, etc.

This comes handy if you want to use devise in your cells, or any other helper method that requires this methods to be defined.

{% highlight ruby %}
Cell::Rack.render_cell_for(:post, :show, controller, controller.request.query_parameters)
{% endhighlight %}

With the last param (controller.request.query_parameters) for the method we are getting any paramter that you can get in the call ('/posts?locale=en') and we pass it to the cell to be able to use it as an argument later in our action.

This performs almost as fast as having a metal (we don´t pass through the application controller). This also places all the necessary logic inside the cell saving us from creating an extra controller in the application to deal with the ajax calls to update the cell.

Simple clean and elegant, and all without loosing the Cell::Rails in the way.
