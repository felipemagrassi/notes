# RSpec benchmark performance - 2023-05-14 23:35:07.107355426 -0300 -03 m=+156.347273287

Using #rspec to #benchmark and increase system #performance

> Unit and Acceptance tests may help you deliver a working software and guard it against regressing into a complete mess. What is not guaranteed is that the code will remain performant
> Use RSpec to test execution speed, Resources usage, Scalability

Import principles:

*   Measure before optimizing
*   Beautiful code doesn't have to be at odds with speed improvements
*   Simpler code tends to be faster

```ruby
require "rspec/core"
require "rspec/autorun"
require "rspec-benchmark"

def clamp(num,min,max)
  [min, num, max].sort[1]
end

def clamp_fast(num,min,max)
  num > max ? max : (num < min ? min : num)
end

RSpec.configure do |config|
  config.include RSpec::Benchmark::Matchers
end

RSpec.describe "#clamp" do 
  it "performs at least 6M iterations per second" do 
    expect { clamp(70,50,60) }.to perform_at_least(6_000_000).ips
  end

  it 'performs at least 2 and half times faster then the old one' do
    expect { clamp_fast(70,50,60) }.to perform_faster_than {
      clamp(70,50,60)
    }.at_least(2.5).times
  end
end
```
