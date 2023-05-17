# Domain Driven Rails - 2023-05-16 21:17:07.851906673 -0300 -03 m=+0.013608244

tags: #book #arkency #fleeting

## Tactical vs Strategic patterns

Tactical focus on 'items' used to build the model. Strategic
focus on concepts, on communication and definition of ubiquitous language

## Bounded Contexts

Bounded Contexts are a way to divide a large domain into smaller parts.
Making it easier to introduce a new developer into a years-old project,
or to introduce a new feature into a large project. Making it easier to
divide and conquer, breaking problems into smaller chunks

As projects grow, they tend to become a Big Ball of Mud. Bounded Contexts
are a way to avoid that. Different groups of people will use different
vocabularies in different parts of a large organization. DDD divides up
a large system into Bounded Contexts, each having a unified model.

## Creating smaller models and clear data ownership per departments

Imagine a system that have many perspectives on the same data, maybe, forecasting,
pricing, inventory, sales, etc. Each of these departments will have a different
perspective on the same data and different needs. DDD creates smaller models
to clear the data ownership per department. Having a Sales::Product, Pricing::Product
Invetory::Product, with their own attributes and methods, and a shared
Product::Product that will be used by all of them. Meaning that a quantity
attribute will be used by Inventory::Product, but not by Sales::Product.
A price attribute can only be changed by Pricing::Product, but not by
Inventory::Product.

## Value Objects

Value Objects are objects with no identify.

Since they have no identity, comparing them is possible

```ruby
Money.new(100, 'BRL') == Money.new(100, 'BRL') # => true
Money.new(100, 'BRL') == Money.new(100, 'USD') # => false
```

They are designed to be immutable and if something changes, a new object is created.
You should not be able to an attribute inside a value object.

```ruby
price = Money.new(100, 'BRL')
price.amount = 110 # => NoMethodError

# Correct way

price = Money.new(100, 'BRL')
price = price + Money.new(10, 'BRL')

price # => Money.new(110, 'BRL')
```

The benefit of using Value Objects is that they are immutable, so you can
pass them around, without passing from method to method or class to class.

And they can protect simple business rules, as:

```ruby
Money.new(1000, 'XXX') # => Money::Currency::UnknownCurrencyError
```

### School Year Example

```ruby

class SchoolYear < Struct.new(:start_year, :end_year)
  def initialize(start_year, end_year)
    raise ArgumentError unless Fixnum === start_year
    raise ArgumentError unless Fixnum === end_year
    raise ArgumentError unless start_year >= 0
    raise ArgumentError unless start_year+1 == end_year
    super(start_year, end_year)
  end
  
  def self.from_name(name)
    start_year, end_year = *name.split("/")
    from_years(start_year, end_year)
  end

  def self.from_years(start_year_string, end_year_string)
    new( Integer(start_year_string), Integer(end_year_string) )
  end

  def self.from_date(date)
    year = date.year
    if date < Date.new(year, 8, 1)
      new(year-1, year)
    else
      new(year, year+1)
    end
  end

  def self.current
    from_date(Date.current)
  end

  def name
    [start_year, end_year].join("/")
  end

  def next
    self.class.new(start_year.next, end_year.next)
  end

  def prev
    self.class.new(start_year.prev, end_year.prev)
  end

  def starts_at
    Date.new(start_year, 8, 1)
  end

  def ends_at
    Date.new(end_year, 7, 31)
  end

  private :start_year=, :end_year=
end

```

Points to consider:

*   Protects simple rules/constraints, 2012/2014 invalid, 2012/2013 valid

Usage in ruby:

```ruby
class Klass
  attr_accessor :school_year # ;)
end

klass = Klass.new
klass.school_year = SchoolYear.from_name("2012/2013")
klass.school_year # => #<struct SchoolYear start_year=2012, end_year=2013>

```

Usage in ActiveRecord:

```ruby

class Klass < ActiveRecord::Base
  def school_year=(sy)
    if sy.nil?
      self['school_year_name']= nil
    else
      self['school_year_name']= sy.name
    end
  end

  def school_year
    name = self['school_year_name']
    return nil if name.nil?
    SchoolYear.from_name(name)
  end
  
  def self.in_year(sy)
    where(school_year_name: sy.name)
  end
end

Klass.first!.school_year
Klass.new.tap{|k| k.school_year = SchoolYear.new(2015, 2015)}
Klass.in_year(SchoolYear.new(2013, 2014))

```

```ruby

class Klass < ActiveRecord::Base
  def self.in_year(sy)
    where(
      school_year_start: sy.start_year, 
      school_year_end: sy.end_year
    )
  end
  
  def school_year=(sy)
    if sy.nil?
      self.school_year_start = self.school_year_end = nil
      return
    end
    self.school_year_start = sy.start_year
    self.school_year_end   = sy.end_year
  end

  def school_year
    return nil if school_year_start.nil? || school_year_end.nil?
    SchoolYear.from_years(school_year_start, school_year_end)
  end
end

Klass.first!.school_year
Klass.new.tap{|k| k.school_year = SchoolYear.new(2015, 2015)}
Klass.in_year( SchoolYear.new(2014, 2015) )

```
