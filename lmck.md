# Testing jsons with rspec - 2023-05-14 23:39:31.095625311 -0300 -03 m=+420.335543182

testing jsons with #rspec

```ruby

RSpec.describe 'json response' do 
let(:response) do {
  person: {
    "id": 1,
    "name": "John Doe",
  },
  "pets": [
    {
      "id": 1,
      "name": "Fido",
      "type": "dog"
    },
    {
      "id": 2,
      "name": "Garfield",
      "type": "cat"
    }
  ]
}
end

it 'gets the right response' do
  expect(response).to match({
    person: {
      "id": 1,
      "name": "John Doe",
    },
    "pets": [
      {
        "id": 1,
        "name": "Fido",
        "type": "dog"
      },
      {
        "id": 2,
        "name": a_string_matching(/gar/i),
        "type": a_string_including("cat")
      }
    ]
  })
  end
end


```



