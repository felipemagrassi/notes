# Ruby Retry - 2023-05-15 00:30:41.193175035 -0300 -03 m=+2818.024947009

When using #retry in #ruby, retry tells the RubyVM to back execution up to the beginning of the nearest `begin/rescue/end` block and try again.

```ruby

require "net/http"
require "webmock"
include WebMock::API

WebMock.enable!

stub_request(:get, "www.example.org")
  .to_raise("Packets devoured by rodents")
  .to_raise("Request saw its shadow")
  .to_return(body: "OK")

def make_request(tries: 3)
  result = Net::HTTP.get(URI("http://www.example.org"))
  puts "Success: #{result}"
rescue => e
  tries -= 1
  puts "Error: #{e}. #{tries} tries left."
  retry if tries > 0
  raise
end

puts make_request
```

## Testing retryable code

```ruby
class Client
  def make_request
    Net::HTTP.get_response("www.rubytapas.com", '/').code
  end
end

require 'delegate'

class RetryingClient < DelegateClass(Client)
  def make_request(*, times: 3)
    super
    rescue
    retry if (times -= 1) > 0
    raise
  end
end

require 'rspec/autorun'

RSpec.describe RetryingClient do 
  it 'delegates requests to an inner client object' do
    inner = double(make_request: 'RESULT')
    client = RetryingClient.new(inner)
    expect(client.make_request).to eq('RESULT')
  end

  it 'retries if a request fails' do 
    inner = double
    client = RetryingClient.new(inner)
    times = 2
    allow(inner).to receive(:make_request) do 
      raise 'Test' if (times -= 1) > 0
      'RESULT'
      end
    expect(client.make_request).to eq('RESULT')
  end

  it 'gives up after 3 tries' do
    inner = double
    client = RetryingClient.new(inner)
    times = 4
    allow(inner).to receive(:make_request) do 
      raise 'Test' if (times -= 1) > 0
      'RESULT'
      end
    expect { client.make_request }.to raise_error('Test')
    expect(inner).to have_received(:make_request).exactly(3).times
  end
end
```
