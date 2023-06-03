# Using rspec tags - 2023-05-14 23:30:19.804232828 -0300 -03 m=+0.010685582

using #rspec tags to filter tests

```ruby

RSpec.configure do |config|
  config.filter_run_including bug: 123

  if Gem::Version.new(RUBY_VERSION) < Gem::Version.new('2.3.0')
    config.filter_run_excluding min_ruby: '2.3.0'
  end

  config.define_derived_metadata(file_path: %r{/spec/integration}) do |metadata|
    metadata[:type] = :integration unless metadata.key?(:type)
  end
end

RSpec.describe 'something' do
  it 'does something', :slow do
    # ...
  end

  it 'does something else', :bug do
    # ...
  end

  it 'does something else with bug', bug: 123 do
    # ...
  end

  it 'does something else with bug', min_ruby: '2.3.0' do
    # ...
  end
end

```

## focus on a test

```ruby
RSpec.configure do |config|
  config.filter_run_when_matching :focus
end

describe "something" do
  it "does something" do
    # ...
  end

  it "does something else", :focus do
    # ...
  end
end


```

```bash
bundle exec rspec --tag slow #run only slow tests
bundle exec rspec --tag '~slow' #run all tests except slow
bundle exec rspec -t bug:123 #run all tests with tag bug:123
```
