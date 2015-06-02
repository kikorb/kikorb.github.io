---
layout: post
title:  "Cells in rails"
date:   2015-05-19 19:59:46
categories: rails cells routes
summary: "When I first tried the gem cells I was engaged with the idea of having components that could be reusable, self aware and isolated from the rest of my apps.
<br/><br/>
The idea is simple. Say you have a partial _sidebar.html that is rendered in
different parts of your website. If this partial needs a couple of instance variables
that had to be calculated the problem begins..."
---

When I first tried the gem cells I was engaged with the idea of having components that could be reusable, self aware and isolated from the rest of my apps.

The idea is simple. Say you have a partial _sidebar.html that is rendered in
different parts of your website. If this partial needs a couple of instance variables
that had to be calculated the problem begins.

# Where do I put this logic?

Thanks to Rails we have many different ways of solving this. The possibilities are endless:

- We could place this logic in a helper method... (Yes I also think it is a good joke). Helpers are not meant to have logic inside (application logic, rendering logic is totally fine). Is never a good idea to access the db or doing heavy calculation inside a helper.

- We could have an action in our controller that gets called with a before_filter to make sure it gets calculated before rendering.

- If this is going to be reused in multiple parts of our app this could live in the application controller, and then we will just need to remember to have a before filter where we need this data.

- Of course having to remember that is not great, and if we continue using this pattern the application controller ends with 100 methods.

- Concerns. Place the method in a concern and even the before filter and then include the concern in the controllers you need it. The problem is that doing include Sidebars in a controller might not be a good idea. Why will the controller need to know about the sidebar in the first place?

# Cells to the rescue

> Cells allow you to encapsulate parts of your page into separate MVC components. These components are called view models.

> You can render view models anywhere in your code. Mostly, cells are used in views to replace a helper/partial/filter mess, as a mailer renderer substitute or they get hooked to routes to completely bypass ActionController.


With the cells everything is solved:
In our layout we will call render_cell(:sidebar, :show) to display the sidebar. Then our cell structure will have a sidebar_cell.rb and a sidebar/ folder. Perfect.

Now we have a place to put our logic (sidebar_cell.rb) and a place for our views (sidebar/).

Cells are getting better and better by the time I write this. There is a big change comming that will depreace Cell::Base and Cell::Rails. If you want to know more about cells check their [github page](https://github.com/apotonick/cells)
