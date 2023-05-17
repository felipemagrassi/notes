# Smaller objects creates faster code - 2023-05-15 00:38:04.873263297 -0300 -03 m=+3261.705035261

tags: #ruby #performance

```ruby
require "csv"
require "sinatra"

data = CSV.table('visitors.csv')

get "/main" do
  data.reduce(0) { |acc, e_record| 
    acc + (e_record[params[:key].to_sym] || 0)
  }.to_s
end

get "/thin" do
  data[params[:key].to_sym].reduce(0) { |acc, e_record| 
    acc + (e_record || 0)
  }.to_s
end
```
