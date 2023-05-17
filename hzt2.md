# ObjectSpace in #ruby - 2023-05-15 00:40:28.947863761 -0300 -03 m=+3405.779635725

tags: #ruby #objectspace
The #count\_objects method returns a hash describing Ruby current memory state. The maximum number of live objects without triggering garbage collection

```ruby
p ObjectSpace.count_objects
```

*Results:* `{:TOTAL=>20023, :FREE=>638, :T_OBJECT=>478, :T_CLASS=>675, :T_MODULE=>33, :T_FLOAT=>4, :T_STRING=>9770, :T_REGEXP=>70, :T_ARRAY=>2069, :T_HASH=>169, :T_STRUCT=>10, :T_BIGNUM=>1, :T_FILE=>26, :T_DATA=>316, :T_MATCH=>22, :T_COMPLEX=>1, :T_SYMBOL=>28, :T_IMEMO=>5675, :T_ICLASS=>38}`

*   Once `ObjectSpace.count_objects[:FREE]` reaches 0, GC is triggered
*   Counts the available space to put objects in

## Creating a simple MemoryProfiler with ObjectSpace

```ruby
class MyMemoryProfiler
  def self.profile
    GC.disable
    before = ObjectSpace.count_objects
    yield
    after = ObjectSpace.count_objects
    after.each { |k,v| after[k] = v - before[k]} # Subtract the number of objects we had before the block from the number we had after 
    after[:T_HASH] -= 1
    after[:FREE] += 1
    after.reject { |k,v| v == 0 }
    ensure
      GC.enable
  end
end

print(
  MyMemoryProfiler.profile do 
    100.times { 'hello' + ' ' + 'world'}
  end) # hello -> "" -> world -> "hello " -> "hello world"
```

*Results:* `{:FREE=>-529, :T_STRING=>527, :T_IMEMO=>2}`

## Using ObjectSpace to print all active objects by class

```ruby
result = ObjectSpace.each_object.
         map(&:class).
         each_with_object(Hash.new(0)) { |e, h| h[e] += 1 }.
         sort_by { |k,v| v }

p result.last(3)
```

*Results:* `[[Class, 366], [Array, 1625], [String, 8250]]`

## Using ObjectSpace to print all ActiveRecord Objects

```ruby
require "active_record" 
require 'pg'

ActiveRecord::Base.establish_connection(
  adapter: 'postgresql',
  database: 'postgres',
  username: 'postgres',
  password: 'postgres',
  host: 'localhost'
)

class Order < ActiveRecord::Base
end

10_000.times { Order.new }
result = ObjectSpace.each_object(ActiveRecord::Base) { |r| r }

p result # Max allocation
```

*Results:* `3872`
