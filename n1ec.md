# Confident Ruby - 2023-05-16 21:17:44.349233259 -0300 -03 m=+36.510934830

tags: #fleeting #book

## Performing with confidence

> Writing a method is like being a captain of a naval vessel, pacing the bridge and issuing orders. A well-trained crew doesn't need to be told what to do, but they do need to know what to do. A well-written method doesn't need to be told what to do, but it does need to know what to do.

**Trusting your methods require three elements:**

*   Identify the messages we want to send in order to accomplish the task
*   Identify the roles which correspond to those messages
*   Ensure method's logic receives objects which can play those roles

### Handling CSV import example

**First we need to identify what need to be done:**

1.  Parse the purchase records from the CSV contained in a provided IO object
2.  For each purchase record, use the record's email address to get the associated customer record, or, if the email hasn't been seen before, create a new customer record
3.  Use the CSV record's product ID to find or create a product record in our system
4.  Add the product to the customer record's list of purchases
5.  Notify the customer of the new location where they can download their files and update their account info
6.  Log the successful import of the purchase record

**Second we need to identify the messages:**

1.  `#parse_csv_purchase_records`
2.  `for #each purchase record, use the #email_address to #find_or_create_customer`
3.  `#find_or_create_product`
4.  `#add_product_to_customer`
5.  `#notify_of_files`
6.  `#log_successful_import`

**Third we need to identify the roles:**

```
| Message                     | Receiver Role     |
| --------------------------- | ----------------- |
| `#parse_csv_purchase_records` | csv\_data\_parser   |
| `#each`                       | purchase\_list     |
| `#email_address, #product_id `| purchase\_record  |
| `#find_or_create_customer`   | customer\_list     |
| `#find_or_create_product`   | product\_inventory |
| `#add_purchased_product`  | customer          |
| `#notify_of_files_available` | customer          |
| `#log_successful_import`| data\_importer     |
```

*parse\_csv\_puchase\_records and csv\_data\_parser is redundant, so we can remove the first one*

**Fourth with our list of roles and messages, we can rewrite our steps:**

1.  csv\_data\_parser.parse\_purchase\_records
2.  for puchase\_list.each purchase\_record use purchase\_record.email\_address to customer\_list.find\_or\_create\_customer
3.  use purchase\_record.product\_id to product\_inventory.find\_or\_create\_product
4.  customer.add\_purchased\_product
5.  customer.notify\_of\_files\_available
6.  data\_importer.log\_successful\_import

```ruby
def import_csv_purchase_data(data)
  purchase_list = csv_data_parser.parse_purchase_records(data)
  purchase_list.each do |purchase_record|
    customer = customer_list.find_or_create_customer(purchase_record.email_address)
    product = product_inventory.find_or_create_product(purchase_record.product_id)
    customer.add_purchased_product(product)
    customer.notify_of_files_available
    log_successful_import(purchase_record)
  end
end
```

> Avoid creating methods for tools, create methods that tells a story. We shouldn't be discussing the ORM, conventions for acessing data... we should be discussing the business logic.

## Collecting Inputs

### Indirect input

**Indirect inputs** are inputs that are returned by another component which is used by the service. Indirect inputs may be actual returns of functions or the state of the system. For example, if a service is using a database, the database is an indirect input. If a service is using a cache, the cache is an indirect input. I/O can be another type of indirect input

**The more levels of indirection that appear in a single method, the more closely that method is tied to the structure of the code around it, and more likely to break if that structure changes**. [Law of Demeter](https://en.wikipedia.org/wiki/Law_of_Demeter)

Being able to trust the inputs of a method to respond to the messages being sent to them, is a key part of writing code that tells a story. Bridging the gap from the objects we have, to the roles we need is

1.  Coerce objects into the roles we need
2.  Reject unexpected values which can't play the role we need (creating object neighbors)
3.  Substitute known-good objects for unacceptable inputs

### Protecting against unexpected inputs

Some languages have a concept of type checking, which is a way of ensuring that the inputs to a method are of the correct type. Ruby does not have this feature, but we can still protect against unexpected inputs using some of ruby built-in conversion protocols `#to_str #to_i #to_path #to_ary`

**Implicit vs Explicit conversions**

**Implicit will try to convert the object to the type we need, and if it can't, it will raise an exception. Explicit will try to convert the object to the type we need, and if it can't, it will return nil.**

Ruby has a number of implicit conversions, which means that ruby will automatically convert an object to another type if it is needed. For example, if you call `#to_s` on an object, ruby will call `#to_str` on the object, and if that fails, it will call `#to_s` on the object. This is a useful feature, but it can also lead to unexpected behavior. For example, if you call `#to_s` on an object, and that object has a `#to_str` method, but that method returns an object that does not respond to `#to_s`, ruby will raise an exception. This is because ruby will try to call `#to_s` on the result of `#to_str`, and if that fails, it will raise an exception.

```ruby

now = Time.now

puts "now responds_to(:to_s) #{now.respond_to?(:to_s)}"
puts "now responds_to(:to_str) #{now.respond_to?(:to_str)}"

puts now.to_s
puts now.to_str # this will raise an exception
```

*Results:*

    /tmp/mdeval//confidentrubymd_93_102.rb:8:in `<main>': undefined method `to_str' for 2023-02-18 17:59:41.393580958 -0300:Time (NoMethodError)
    Did you mean?  to_s
                   to_r
    now responds_to(:to_s) true
    now responds_to(:to_str) false
    2023-02-18 17:59:41 -0300

Ruby uses `#to_str` to verify that an object can be converted to a string. If an object does not respond to `#to_str`, ruby will raise an exception. This is a good thing, because it means that we can be sure that an object can be converted to a string, and we can be sure that the result of that conversion will be a string.

```ruby
class ArticleTitle
  def initialize(text)
    @text = text
  end
end

title = ArticleTitle.new("Hello World")
begin 
  puts "Today's article title #{'is + ' + title}"
rescue => e
  puts e
end

class CorrectArticleTitle
  def initialize(text)
    @text = text
  end

  def to_str
    @text
  end
end

title = CorrectArticleTitle.new("Hello World")
puts "Today's article title #{'is + ' + title}"
```

*Results:*

    no implicit conversion of ArticleTitle into String
    Today's article title is + Hello World

Ruby core library doesn't use explicits conversions, so those using ruby core library can be sure that the objects they are using will respond to the messages they are sending to them. If you are using a gem, you should check the documentation to see if the gem uses explicit conversions.

```
| Method     | Target Class | Type     |
| ---------- | ------------ | -------- |
| `#to_ary`    | Array        | Implicit |
| `#to_a`      | Array        | Explicit |
| `#to_c`      | Complex      | Explicit |
| #to\_enum`   | Enumerator   | Explicit |
| #to\_h      | Hash         | Explicit |
| #to\_hash   | Hash         | Implicit |
| #to\_i      | Integer      | Explicit |
| #to\_int    | Integer      | Implicit |
| #to\_io     | IO           | Implicit |
| #to\_open   | IO           | Implicit |
| #to\_path   | String       | Implicit |
| #to\_str    | String       | Implicit |
| #to\_s      | String       | Explicit |
| #to\_proc   | Proc         | Implicit |
| #to\_r      | Rational     | Explicit |
| #to\_regexp | Regexp       | Implicit |
| #to\_sym    | String       | Implicit |
```
### Explicit Conversion example

```ruby
PHONE_EXTENSIONS =["Operator", "Sales", "Customer Service"]
def dial_extension(dialed_number)
  dialed_number = dialed_number.to_i
  extension = PHONE_EXTENSIONS[1]
  puts "Please hold while you are connected to #{extension}"
end
nil.to_i                        # => 0
dial_extension(nil)
# >> Please hold while you are connected to Operator
```

*Results:* `Please hold while you are connected to Operator`

### Implicit conversion example

```ruby
def set_centrifuge_speed(new_rpm)
  new_rpm = new_rpm.to_int
  puts "Adjusting centrifuge to #{new_rpm} RPM"
end
bad_input = nil
set_centrifuge_speed(bad_input)
```

*Results:*

    /tmp/mdeval//confidentrubymd_194_201.rb:2:in `set_centrifuge_speed': undefined method `to_int' for nil:NilClass (NoMethodError)
    	from /tmp/mdeval//confidentrubymd_194_201.rb:6:in `<main>'

### Conditionally calling examples

**bad examples:**

```ruby
def my_open(filename)
  raise TypeError unless filename.is_a?(String)
  # ...
end

class String
  def to_path
    self
  end
end

def my_open(filename)
  filename = filename.to_path
  # ...
end
```

**This examples are bad because they limit the method we are building or they change the behavior of a core class, just to make our code work.**

**good example:**

```ruby
def my_open(filename)
  filename = filename.to_path if filename.respond_to?(:to_path)
  filename = filename.to_str
  # ...
  return puts "file opened"
end

my_open("file.txt")
```

*Results:* `file opened`

This checks both String and Pathname objects, and it doesn't change the behavior of the String class, while also blocking other objects from being passed to the method.
