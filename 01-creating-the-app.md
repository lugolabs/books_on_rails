# 1. The App

In this series, called *Building a Ruby on Rails app*, we'll create a web app that lists books about Ruby on Rails. It should also record the sales rank of the book daily (source from Amazon.com), and show the ranking movement. You can see the finished live site at [Books on Rails website](http://lugolabs.com/booksonrails).

Our web app is called *Books on Rails*, so let's create a Rails app with that name:

```sh
rails new booksonrails
```

Isn't that exciting! The familiar command creates a nice new Rails web app for us, ready to be filled with the functionality we have planned.

Let's dive straight into it, by creating the book model.

Book model
--

We want to show a few details about our books: title, author, publication date, and an image. We also want to display a movement on the books' sales rankings. Let's add them to the model generator:

```sh
bundle exec rails g model book title:string author:string asin:string published_at:string amazon_image_url:string image_url:string previous_sales_rank:integer last_sales_rank:integer
```

And create the table on the database:

```sh
bundle exec db:migrate
```

Books controller
--

We will create books data via a script that talks to Amazon API, so our controller will be quite small, with just one action:

```sh
bundle exec rails g controller books index --skip-javascripts
```

This will create an empty controller with only the `index` action and also the `index.html.erb` view. (We skip the creation of JavaScript files, as we will not use JavaScript in this app, at least not at the start.)

We take the chance to setup our routes here:

```ruby
# config/routes.rb

Rails.application.routes.draw do
  resources :books, :only => :index
  root 'books#index'
end
```

It's important we create only the routes we need, for faster caching, less testing, less code generated, etc..

We'll create a controller test as well:

```ruby
# test/controllers/books_controller_test.rb

class BooksControllerTest < ActionController::TestCase
  test "should get index" do
    get :index
    assert_response :success
    assert_not_nil assigns(:books)
  end
end
```

and run the test:

```sh
bundle exec rake
```

The test should fail, complaining that our *books* is actually nil. So let's fix that:

```ruby
# app/controllers/books_controller.rb

class BooksController < ApplicationController
  def index
    @books = Book.order(:last_sales_rank)
  end
end
```

This will allow us to show the books order by the highest ranked. If we run the test now, it should pass.

Please note that the command `bundle exec rake` will run all our tests as it defaults to `bundle exec rake test`. If we want to run only the controller test, we can use the command `bundle exec rake test:controllers`.


Application views
--

We already have the books' `index.html.erb` view created by the generator, and the `application.html.erb` layout view. Let's change the latter so it looks like this:

```html
<!DOCTYPE html>
<html>
<head>
  <title>Books on Rails</title>
  <%= stylesheet_link_tag 'application', media: 'all' %>
  <%= csrf_meta_tags %>
</head>
<body>

  <%= render "layouts/header" %>

  <section class="app-cont">
    <%= yield %>
  </section>

  <%= render "layouts/footer" %>

</body>
</html>
```

We remove the JavaScript link as we won't use it in this app. We also remove any reference to *turbolinks* in the stylesheet link, for the same reason. Let's also remove the *turbolinks* gem from the Gemfile, as it is important to have only the gems we need there. Then run the `bundle` command to clean up the gems.

As we saw with our tests earlier, I like to use the conventions that come with Ruby on Rails, as they do a great job to have a full, well structured, and safe application.

Going back to our *application* layout, we have added a header, a footer, and wrapped the main `yield` inside a `div` with an `app-cont` class.

It's important we create these building blocks (header, and footer) into separate partials. This way we can keep our layout clean, and tidy, and organize our code better.

```html
<!-- app/views/layout/_header.html.erb -->

<% cache do %>
  <header class="app-header">
    <div class="img"></div>
    <div class="app-cont">
      <h1 class="app-logo">
        <%= link_to :root do %>
          Books <b>on</b> Rails
        <% end %>
      </h1>
    </div>
  </header>
<% end %>
```

We use the HTML5 semantic markup to create the header, and fill it with a logo and other elements we'll style later. We also wrap the html inside a `cache` call, so that the HTML is cached on subsequent requests.

The footer is similar:

```html
<!-- app/views/layout/_footer.html.erb -->

<% cache do %>
  <footer class="app-footer">
    <div class="img"></div>
    <div class="app-cont">
      <h1 class="app-logo">
        <%= link_to :root do %>
          Books <b>on</b> Rails
        <% end %>
      </h1>

      <div class="copy">
        <%= Time.now.year %> &copy; <%= link_to 'Lugo Labs', 'http://www.lugolabs.com' %>.
        Data by <%= link_to 'Amazon.com', 'http://www.amazon.com' %>.
      </div>
    </div>
  </footer>
<% end %>
```

If we start the development server now, we can see the header and footer in action, albeit not styled.

Conclusion
--

In this tutorial we specified the requirements of our simple web app, we created the app, generated the first model, controller and views. At the end we added a simple header and footer with a logo.

In the [second part](/lugoland/articles/76-building-a-ruby-on-rails-app-part-2-styling-the-layout) of this series will add a paint of style to our views.

* [Books on Rails](http://lugolabs.com/booksonrails) live example
