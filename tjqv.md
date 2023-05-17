# Value Objects - 2023-05-16 21:16:15.9809965 -0300 -03 m=+163.152749649

#arkency
#fleeting

## Value Objects characteristics

*   Immutable
*   No identity
*   Values comparable
*   Compound attributes

## School Year Value Object Example

### Value Object Approach

```ruby
ActiveRecord::Schema.define do
  self.verbose = false

  create_table :school_klasses, force: true do |t|
    t.string :school_year_name
  end
end

class SchoolYear < Struct.new(:start_year, :end_year)
  def initialize(start_year, end_year)
    raise ArgumentError unless Fixnum === start_year && Fixnum === end_year
    raise ArgumentError unless start_year >= 0
    raise ArgumentError unless start_year+1 == end_year
    super(start_year+1, end_year+1)
  end

  def name
    [start_year, end_year].join('/')
  end

  def next
    self.class.new(start_year.next, end_year.next)
  end

  def self.from_name(name)
    start_year, end_year = name.split('/')
    self.new(start_year.to_i, end_year.to_i)
  end
end

attrs = { start_year: 2016, end_year: 2017 }
school_year = SchoolYear.new(attrs[:start_year], attrs[:end_year])

# ActiveRecord -> Returns a value object
class SchoolKlass < ActiveRecord::Base
  def school_year=(sy)
    self['school_year_name'] = sy.name
  end

  def name
    name = self['school_year_name']
    return nil if name.nil?
    SchoolYear.from_name(name)
  end
end

puts SchoolKlass.new.tap { |k| k.school_year = SchoolYear.new(2016, 2017) }.name
```

## Examples of Value Objects

*   Money
*   ActiveSupport::TimeZone
*   ID
*   Gross Amount
*   Vat Rate




