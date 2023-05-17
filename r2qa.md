# Caller-Specified fallback in #ruby - 2023-05-15 00:36:35.758798801 -0300 -03 m=+3172.590570765

Creating fallbacks for methods with a block parameter in #ruby

```ruby
class TemperatureError < StandardError; end

def get_temp(query, &qfallback)
  fallback ||= ->(error) { raise } 
  key  = ENV['WUNDERGROUND_KEY']
  url  = "http://api.wunderground.com/api/#{key}/conditions/q/#{query}.json"
  data = open(url).read
  JSON.parse(data)['current_observation']['temp_f']
rescue => error
  fallback.call(error)
end

puts get_temp('00000') { "N/A" }
puts get_temp('00000') { TemperatureError.new('No temperature') }
```

## Testing Fallbacks

```ruby
def get_temp(query, &fallback)
  fallback ||= ->(error) { raise } 
  key  = ENV['WUNDERGROUND_KEY']
  url  = "http://api.wunderground.com/api/#{key}/conditions/q/#{query}.json"
  data = open(url).read
  JSON.parse(data)['current_observation']['temp_f']
rescue => error
  fallback.call(error)
end

require 'rspec/autorun'

describe 'get_temp' do
  it 'returns the fallback value' do
    expect(get_temp('00000') { 'N/A' }).to eq('N/A')
  end

  it 'raises the fallback error' do
    expect { get_temp('00000') { raise 'No temperature' } }.to raise_error('No temperature')
  end

  it 'tests using another value' do
    yielded = false
    get_temp('00000') { |error| yielded = true }
    expect(yielded).to be true
  end
end
```
