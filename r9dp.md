# Default uuid for rails projects - 2023-05-15 00:26:40.806301157 -0300 -03 m=+2577.638073121

Enable pgcrypto extension for UUIDs in #rails projects

```ruby
class EnableUUID < ActiveRecord::Migration
  def change
    enable_extension 'pgcrypto'
  end
end
```

Add the following to your `config/initializers/generators.rb`

```ruby
Rails.application.config.generators do |g|
  g.orm :active_record, primary_key_type: :uuid
end
```

Run the migration

```bash
rake db:migrate
```
