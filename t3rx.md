# Building a safe proxy with #ruby - 2023-05-15 00:02:33.619457707 -0300 -03 m=+1130.451229671

Building a gateway within external services can create a snowball effect of exceptions being raised and breaking part of the application, building a proxy can help if you need to
assert that something is working after or before the primary logic of the class

```ruby
require 'logger'
require 'delegate'

class PostGateway
  def initialize(session, options = {})
    @session = session
    @logger = options.fetch(:logger) { ::Logger.new($stderr) }
  end

  def post(*args)
    10
  end
end

class PostGatewayProxy < DelegateClass(PostGateway)
  def initialize(target, on_error: ->(*){})
    super(target)
    @error_handler = on_error
  end

  def post(*)
    super
  rescue StandardError
    @error_handler.call
    {
      success: false,
      status: 404,
      test: 'test'
    }
  end
end

require 'rspec'
require 'rspec/autorun'

RSpec.describe PostGatewayProxy do
  it 'delegates #post' do 
    gw = instance_spy(PostGateway)
    gwp = described_class.new(gw)

    gwp.post
    expect(gw).to have_received(:post).once
  end

  it 'passes arguments through to #post' do
    gw = instance_spy(PostGateway)
    gwp = described_class.new(gw)

    arg1, arg2 = double('arg1'), double('arg2')
    gwp.post(arg1, arg2)

    expect(gw).to have_received(:post).with(arg1, arg2).once
  end

  it 'prevents standard exceptions from exiting' do
    gw = instance_spy(PostGateway)
    gwp = described_class.new(gw)

    allow(gw).to receive(:post).and_raise(StandardError)
    expect { gwp.post }.not_to raise_error
  end

  it 'returns default error message' do 
    gw = instance_spy(PostGateway)
    gwp = described_class.new(gw)
    hash = { success: false }

    allow(gw).to receive(:post).and_raise(StandardError)
    expect(gwp.post).to match(hash_including(hash))
  end

  it 'calls the :on_error handler in :post' do
    gw = instance_spy(PostGateway)
    handler = double("handler", call: nil)
    gwp = described_class.new(gw, on_error: handler)
    allow(gw).to receive(:post).and_raise(StandardError)

    gwp.post 
    expect(handler).to have_received(:call)
  end
end

puts PostGatewayProxy.new(PostGateway.new(1), on_error: ->(*) {   }).post
```



