# Using benchmark gem in ruby - 2023-05-15 00:01:35.112695544 -0300 -03 m=+1071.944467508

#ruby #benchmark #performance

> Premature optimization is the root of all evil - Donald Knuth (ka-nuf)

*   Run a code a bunch of time and see how long took to finish or how much memory took to complete task

**wallclock time:** how long the operation took in real time

```ruby
start_time = Time.now

1_000_000.times do 
  2 + 2
end

result = Time.now - start_time
puts result
```

*Results:* `0.024850994`

## Using the ruby benchmark standard library

```ruby
require 'benchmark'

result = Benchmark.measure do 
  1_000_000.times do 
    2 + 2
  end
end

puts result.inspect
```

*Results:* `#<Benchmark::Tms:0x0000556ab0013ec8 @label="", @real=0.02335214599952451, @cstime=0.0, @cutime=0.0, @stime=0.00041300000000000017, @utime=0.022936, @total=0.023349>`

| Column | Meaning                                       |
| ------ | --------------------------------------------- |
| real   | all the time spent in code (wallclock time)   |
| stime  | all the time spent in system (IO/network)     |
| utime  | time spent in user land code (RVM not kernel) |
| total  | all the time spent in code                    |

**Dont assume without generating evidence of real optimization problems!! In this example, optimizing the reverse of the string is worthless compared to the IO time. Only optimize real consuming problems**

```ruby
require 'benchmark'

result = Benchmark.measure do 
  10_000.times do 
    str = 'SUPERCALIFRAGILISTICEXPIALIDOCIOUS'
    str.reverse!
    File.write('word.txt', str)
  end
end

puts result.inspect
```

*Results:* `#<Benchmark::Tms:0x0000561f805e4a70 @label="", @real=0.26699286099756137, @cstime=0.0, @cutime=0.0, @stime=0.172765, @utime=0.032594000000000005, @total=0.205359>`

**Comparing results with bmbm:**

*   Using bm x bmbm: avoid cold-started code utilizing bm

```ruby
require 'benchmark'

class Handler
  def handle_stepintime
  end
end

handler = Handler.new
n = 1_000_000

Benchmark.bmbm(16) do |r|
  event = :stepintime

  r.report("case dispatch") do
    n.times do 
      case event
      when :stepintime then handler.handle_stepintime
      end
    end
  end

  r.report("dynamic dispatch") do 
    n.times do 
      handler.public_send("handle_#{event}")
    end
  end
end
```

*Results:*

    Rehearsal ----------------------------------------------------
    case dispatch      0.041832   0.000217   0.042049 (  0.042052)
    dynamic dispatch   0.229858   0.000000   0.229858 (  0.229864)
    ------------------------------------------- total: 0.271907sec

                           user     system      total        real
    case dispatch      0.044031   0.000000   0.044031 (  0.044031)
    dynamic dispatch   0.228426   0.000000   0.228426 (  0.228432)

## Benchmark-IPS

*   Benchmark requires a good number of repetitions through trial and error, Benchmark-IPS solves that
*   Benchmark `#bmbm` requires a good number of columns to be perfectly aligned, Benchmark-IPS solves that
*   Benchmark results are not sorted during runtime, Benchmark-IPS solves that

```ruby
require 'benchmark/ips'

class Handler
  def handle_stepintime
    1 + 1
  end
end

handler = Handler.new

Benchmark.ips do |r|
  event = :stepintime

  r.report("case dispatch") do
    case event
    when :stepintime then handler.handle_stepintime
    end
  end

  r.report("dynamic dispatch") do 
    handler.public_send("handle_#{event}")
  end

  r.compare!
end
```

*Results:*

```
Warming up --------------------------------------
       case dispatch     1.886M i/100ms
    dynamic dispatch   434.771k i/100ms
Calculating -------------------------------------
       case dispatch     18.887M (± 0.2%) i/s -     96.192M in   5.092895s
    dynamic dispatch      4.301M (± 0.4%) i/s -     21.739M in   5.054546s

Comparison:
       case dispatch: 18887487.2 i/s
    dynamic dispatch:  4300870.8 i/s - 4.39x  (± 0.00) slower

```

IPS also gives a `#compare!` method, to easily compare reports, one problem that Benchmark can solve and IPS cant is given the system/real/user time, comparing problems within application
