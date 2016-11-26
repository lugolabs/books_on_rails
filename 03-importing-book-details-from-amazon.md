# 3. Importing book details from Amazon

In [part 2](https://rtsinani.gitbooks.io/books-on-rails/content/02-styling-the-layout.html) we added some necessary stylesheets to our website. In part 3 we're going to the back end again to fill our database with books. For this, I went to the [Amazon website](http://www.amazon.com) and searched for Ruby on Rails books. I extracted a dozen of ASINs (the Amazon book identifier) to use as our sample dataset.

Rake
--

Let's start by creating a Rake file in our tasks folder:

```ruby
# lib/tasks/books.rake

namespace :booksonrails do
  desc 'Import books'
  task :import => :environment do
  end

  def asins
    %w(0321944275 1937785564 1593275722 0134077709 1617291099 B00QW597D8 0692364218 1937785556 B00QK2T1SY 1941222196 B00YPU5MGS B0127BVV8Y 0321659368 B012O6PJMG B012BB0G2W 1491910852 B00NKML6JE 1491054484)
  end
end
```

We have name spaced our tasks to differentiate them from Rails' built-in tasks, thus avoiding any naming conflicts. We also created an empty task, *import*, which we'll use to import the books, by calling it on the command line:

```sh
bundle exec rake booksonrails:import
```

At the end we added the ASINs extracted from Amazon into an array. Now time to go through them and fetch the details:

```ruby
# lib/tasks/books.rake

task :import => :environment do
  start = Time.now
  puts 'Start'
  asins.each do |asin|
    puts "import #{asin}"
    AmazonProxy.import_book asin
  end
  puts "Complete in #{Time.now - start}s"
end
```

We start by adding some messages about when our task start and completes, so that we can have some immediate feedback about our running task. I find it useful to also know the time that tasks take, so that I can improve on the performance if necessary.

As we enumerate through the book ASINs, we make a call to a `AmazonProxy` class to import the book details based on its ASIN. Let's create that class next.


Amazon Ecs
--

Amazon provides a great [API](https://aws.amazon.com) about its items and their sales. In order to access the API we need to open a free merchant account.

Once the account is created, we need to extract the associate tag, the Access Key Id and Secret Key of their web services. We'll use them in a bit.


Amazon Proxy
--

There are plenty of libraries for any programming language that query the API, including ruby. We'll use the [amazon-ecs](https://github.com/jugend/amazon-ecs) gem, so let's add it to our Gemfile:

```ruby
# Gemfile

gem 'amazon-ecs'
```

and run

```sh
bundle install
```

Let's create a proxy class that will interact with the Amazon Web Service (AWS):

```ruby
# app/models/amazon_proxy.rb

class AmazonProxy
  Amazon::Ecs.configure do |options|
    options[:AWS_access_key_id] = 'YOUR ACCESS KEY ID'
    options[:AWS_secret_key]    = 'YOUR SECRET KEY'
    options[:associate_tag]     = 'YOUR ASSOCIATE TAG'
  end
end
```

We also added the keys extracted previously to the `Amazon::Ecs` configuration block. The *amazon-ecs* wraps all the necessary call to Amazon, and you are free to check them out, but we'll use only one here:

```ruby
# app/models/amazon_proxy.rb

class AmazonProxy
  private
    def self.fetch(asin)
      result = Amazon::Ecs.item_lookup(asin, { :response_group => 'Medium' })
      result.items[0]
    end
end
```

The `item_lookup` accepts an ASIN and an options Hash, in which we identify the *response_group* to be *Medium*. This will give us with all the book details we needs, such as title, author, or image.

The result contains an array of items, of which we extract the first one.

Now it's time to write the *import_book* method we called from the Rake task:

```ruby
# app/models/amazon_proxy.rb

def self.import_book(asin)
  item = fetch(asin)
  save_book asin, item
end
```

First, we get the response item by calling the `fetch` method, then we use that to save the book in the following private method:

```ruby
# app/models/amazon_proxy.rb

def self.save_book(asin, item)
  return if Book.exists?(asin: asin)

  options                    = {}
  image_url                  = item.get_hash('MediumImage')['URL']
  item_attributes            = item.get_element('ItemAttributes')
  options[:asin]             = asin
  options[:title]            = item_attributes.get('Title')
  options[:published_at]     = item_attributes.get('PublicationDate')
  options[:author]           = item_attributes.get_array('Author').join(', ')
  options[:amazon_image_url] = image_url
  options[:image_url]        = save_image(asin, image_url)
  Book.create! options
end
```

We start by making sure a book with the same ASIN doesn't exist. Then we extract the details by using different response elements and methods, such as `item_attributes` for the title, publication date and author, or `get_hash` for the image. At the end we save the book into the database. But not before we have saved the image.

Image
--

We could link to the book image on the Amazon website, but for this project we'll store it into our servers. For this, we will use the [rmagick](https://rubygems.org/gems/rmagick) gem, so let's add it the Gemfile:

```ruby
# Gemfile

gem 'rmagick'
```

and run

```sh
bundle install
```

Then we write the private method below:

```ruby
# app/models/amazon_proxy.rb

def self.save_image(asin, image_url)
  relative_path = "/static/#{asin}.jpg"
  file_path     = Rails.root.join("public#{relative_path}")
  Magick::Image.read(image_url).first.resize_to_fill(100, 100).write(file_path)
  relative_path
end
```

First we need to create the *static* folder inside the *public* directory of our app. We use the `Rails.root` capabilities to figure out the full path of the new file, which will be named with book ASIN. *RMagic* will then read the image from the Amazon CDN, and save into our location. We also resize the image to 100x100px, for consistency.

At the end we return the relative path, which we save with the book details, and which will be used to display the image on the web page.

If we run the Rake task on the command line now:

```sh
bundle exec rake booksonrails:import
```

we can see how the book details and image are being saved into our database and server.

Conclusion
--

In this part of *How to create a Rails app* series, we imported book details from AWS. In the [next part](https://rtsinani.gitbooks.io/books-on-rails/content/04-importing-book-sales-from-amazon-with-ruby-on-rails.html) we'll import the book sales from Amazon.

* [Books on Rails](/booksonrails) live example
