# Testing blocks in rspec - 2023-05-14 23:37:28.620993515 -0300 -03 m=+297.860911366

testing blocks in #rspec

```ruby
require 'rspec/autorun'

def find_email_addressses(input)
  input.scan(/[A-Z0-9+_.-]+@[A-Z0-9.-]+/i) do |match|
    yield match
  end
end

describe 'find_email_addresses' do
  it 'yields when it finds an email address' do
    input = 'felipemagrassi@gmail.com'

    expect { |b| find_email_addressses(input, &b) }.to yield_control
  end

  it 'does not yield when it does not find an email address' do
    input = 'felipemagrassi'

    expect { |b| find_email_addressses(input, &b) }.not_to yield_control
  end

  it 'yields the email address when it finds one' do
    input = 'aaaa felipemagrassi@gmail.com'

    expect { |b| find_email_addressses(input, &b) }.to yield_with_args('felipemagrassi@gmail.com')
  end

  it 'yields multiple email addresses when it finds them' do
    input = 'aaaa felipemagrassi@gmail.com teste@gmail.com bbbb'

    expect { |b| find_email_addressses(input, &b) }.to yield_successive_args(
      'felipemagrassi@gmail.com',
      'teste@gmail.com'
    ) 
  end
end
```




