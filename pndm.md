# RSpec boilerplate config - 2023-05-14 23:26:22.148658885 -0300 -03 m=+0.012582059

```
rspec --require rails_helper --format documentation
```

```ruby
# spec/rails_helper.rb
ENV['RAILS_ENV'] ||= 'test'
require File.expand_path('../config/environment', __dir__)
require 'rspec/rails'

RSpec.configure do |config|
  config.infer_spec_type_from_file_location!
end
```

```ruby
# spec/system/home_spec.rb

RSpec.describe 'The home page' do
  before { visit root_url }

  it 'displays a link to log in' do
    expect(page).to have_link('Log in with your email')
  end
end
```
#rspec #boilerplate #config
