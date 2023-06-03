# Streaming globs in ruby - 2023-05-15 00:39:22.149132125 -0300 -03 m=+3338.980904089

tags: #ruby #performance 

```ruby
require 'csv'

def memstats
  size = `ps -o size= #{$$}`.strip.to_i
  "Size: #{size}"
end

puts memstats
visitors = CSV.read('visitors.csv', headers: true)
puts visitors.count { |v| v['Variable_code'] =~ /H01/ }
puts memstats
```

**Huge memory usage due to creating and retaining the object**

```ruby
require 'csv'

def memstats
  size = `ps -o size= #{$$}`.strip.to_i
  "Size: #{size}"
end

puts memstats
count = 0
CSV.foreach('visitors.csv', headers: true) do |v|
   count += 1 if v['Variable_code'] =~ /H01/
end
puts count
puts memstats
```

## Implementing with enumerator

```ruby
require 'csv'

def memstats
  size = `ps -o size= #{$$}`.strip.to_i
  "Size: #{size}"
end

puts memstats
CSV.open('visitors.csv', headers: true) do |csv|
  visitors = csv.each # => Generates a Enumerator when passing w/out blocks
  puts visitors.count { |v| v['Variable_code'] =~ /H01/}
end
puts memstats
```
