# Activerecord GC usage - 2023-05-15 00:29:17.747146262 -0300 -03 m=+2734.578918226

tags: #ruby #performance #rails 

How to use GC with activerecord.

```ruby
require "active_record"
require "securerandom"

ActiveRecord::Base.establish_connection adapter: "sqlite3", database: ":memory:"
ActiveRecord::Migration.verbose = false

ActiveRecord::Schema.define do 
  create_table :users, force: true do |t|
    t.string "name"
    t.string "email"
    t.timestamps null: false
  end
end

class User < ActiveRecord::Base; end

20.times do 
  User.create!(name: SecureRandom.uuid, email: SecureRandom.uuid)
end

def count_allocations
  x = GC.stat(:total_allocated_objects)
  yield
  GC.stat(:total_allocated_objects) - x
end

user = User.first
p count_allocations { user.id }
p user.inspect
p count_allocations { user.id }
```

*Results:*

    6
    "#<User id: 1, name: \"0976b083-6784-4922-bca4-ead2a132e1fb\", email: \"fd72b9e4-2243-4ca4-98de-e644a5bafb97\", created_at: \"2023-01-29 03:13:24.451226000 +0000\", updated_at: \"2023-01-29 03:13:24.451226000 +0000\">"
    0
