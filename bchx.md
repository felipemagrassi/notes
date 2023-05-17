# understanding rspec matchers - 2023-05-14 23:38:26.702929592 -0300 -03 m=+355.942847443

* #rspec creates a object called example for every `it` block and an example group for every `describe` block.

```ruby
RSpec.describe 'an example group' do
  it 'an example' do
    result = 2 + 2
    expect(result).to eq(4)
  end
end

RSpec.describe('another example group') do
  it('another example') do
    result = 2 + 2
    self.expect(result).to(eq(4))
  end
end
```

```ruby
expect(4).to eq(4)
<ExpectationTarget<4>>.to(eq(4))
<ExpectationTarget<4>>.to(Eq<4>)
<Eq<4>.matches?(4)
```
