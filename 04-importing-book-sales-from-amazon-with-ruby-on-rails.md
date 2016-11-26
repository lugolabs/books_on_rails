# Importing book sales from Amazon with Ruby on Rails

In [part 3](/lugoland/articles/77-building-a-ruby-on-rails-app-part-3-importing-book-details-from-amazon) we filled our database with a dozen of Ruby on Rails books with details taken from Amazon Web Services (AWS). We also fetched the book image and stored it into our own servers. In this part, we'll go back to AWS and fetch the book sales.

Book Sale Model
---

We need a model to store the sales rank we fetch from Amazon, so let's generate that now:

```sh
bundle exec rails generate model book_sale book:belongs_to sales_rank:integer
```

For each book we save the sales rank on the day we fetch it from Amazon. Let's do that next.

Amazon Proxy
--

In [part 3](/lugoland/articles/77-building-a-ruby-on-rails-app-part-3-importing-book-details-from-amazon) we created an `AmazonProxy` class to communicate with AWS. We'll add a few methods to that class to get the sales rank information.

```ruby
# app/models/amazon_proxy.rb

def self.import_sale(asin)
  item = fetch(asin)
  save_sale asin, item
end

private

  def self.save_sale(asin, item)
    book = Book.find_by(asin: asin)
    existing = book.book_sales.where('DATE(created_at) = ?', Time.now.to_date).first
    if existing.nil?
      sales_rank = item.get('SalesRank')
      book.book_sales.create(sales_rank: sales_rank)
      book.update_sales_rank_movement
    end
  end
```

First we use our `fetch` method we created on [the previous post](/lugoland/articles/77-building-a-ruby-on-rails-app-part-3-importing-book-details-from-amazon) to grab the book details by the book's unique identifier (ASIN). Then (in the `save_sale` method) we check that we don't have a ranking for the book on the same day. If we don't, we grab the `SalesRank` item attribute from AWS and create a new sale record for the book in the data store. At the end, we update the sales movement of the book. For that we create an instance method on the `Book` model:

```ruby
def update_sales_rank_movement
  recent_sales = sales.last(2)
  return if recent_sales.size == 0
  options = { previous_sales_rank: 0, last_sales_rank: recent_sales[0].sales_rank }
  options[:previous_sales_rank] = recent_sales[1].sales_rank if recent_sales.size > 1
  update options
end
```

This method is simple: we get the last two sales for the book and update the `previous_sales_rank` and 'last_sales_rank' with the appropriate record. These two fields will then be used by the `sales_rank_movement` method:

```ruby
def sales_rank_movement
  @sales_rank_movement ||= (last_sales_rank || 0) - (previous_sales_rank || 0)
end
```

As we see above, the movement shows the difference between the last sale and the one prior to that.


Rake
--

Next, let's create the Rake task that will populate our database everyday.

```ruby
desc 'Import sales ranks'
task :sales => :environment do
  Book.all.each do |book|
    AmazonProxy.import_sale book.asin
  end
end
```

No much mystery here, we navigate through the books and call the `AmazonProxy` method to import the book's sale.

Conclusion
--

In this part of *How to create a Rails app* series, we imported the daily book sales from Amazon and updated our book records. This allows us to show the sales movements daily. In the next part, we'll build the page that shows the books and their sales ranking.

Check [Books on Rails](/booksonrails) for the live example.
