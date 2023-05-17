# Rails performance tips by Nathan - 2023-05-15 00:33:41.376827207 -0300 -03 m=+2998.208599171

We can use profilers and tools to find bottlenecks in our code.
To increase #performance, we first #benchmark our code, then we find the bottlenecks and
then we fix them.

## Memory profiling motivation

Ruby is a language with automatic memory management. So, memory lifecycle of a object isn't something ruby developers have to worry about. Ruby runtime takes care of allocating space for us and alocate that space for other purposes when we're done with the object.
This feature can be a double-edged sword, because when abstractions start leaking, Rubyists without experience in other languages don't really know what to do about it, because you didn't had to think about memory until now

## Always generate assumptions

*   **Never make assumptions of non-benchmarked code**

```ruby
require "benchmark/ips"
index = 42
page_size = 10

Benchmark.ips do |x| 
  x.report('operators') do |times|
    1.upto(times) do 
      index / page_size
      index % page_size
    end
  end

  x.report('divmod') do |times|
    1.upto(times) do
      index.divmod(page_size)
    end
  end

  x.compare!
end
```

*Results:*

```
Warming up --------------------------------------
           operators     3.170M i/100ms
              divmod     2.010M i/100ms
Calculating -------------------------------------
           operators     31.878M (± 2.5%) i/s -    161.658M in   5.074605s
              divmod     19.915M (± 1.0%) i/s -    100.475M in   5.045690s

Comparison:
           operators: 31877962.0 i/s
              divmod: 19915046.8 i/s - 1.60x  (± 0.00) slower

```

## Installing rails profilers

gem 'rack-mini-profiler'
gem 'stackprof'
gem 'flamegraph'

## Always reload the page a couple of time to warm up the app

Application is filling the caches, compiling, etc...

## Motivations for Development slower than production

*   Extra work being done in development environment
*   Processor single-core speed slower (Probably not because of M1, I9 single thread clock speeds compared to AWS M5)
    *   *Motivation to get better performance -> your production machine is probably slower than your development machine*

## Watch for partials and layouts rendering

**Expect a rails controller action to render in =~ 200ms**

## Tracing exceptions

**Check if exceptions are being used as a conditional**

## Catch and Throw vs Exceptions

**Raising is slow than catch and throw**

### Rack mini profiler query param

**localhost:3000/.../?pp=help**

## Flamegraphs and stacks traces

**AKA Sampling Profiler**

## Memory analysis and string counting

## SQL diagnostics and conclusion

*   **Focus on reducing the number of queries**
*   **Focus on reusing already owned information**

## Best performance advices are simple

*   Always index your foreign keys (references - rails)
*   Always load associations before using
*   [Use the Index, Luke!](https://use-the-index-luke.com/)
*   Use Analyse in Postgres
