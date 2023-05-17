# Verifying data integrity in #rails  - 2023-05-15 00:27:31.400672351 -0300 -03 m=+2628.232444305

When updating models without validating then, we can create out of sync data in our rails apps, examples such:

1.  update object attribute using `update_attribute` or `update_column` without validating the object
2.  delete a parent object without deleting its children
3.  persisting an object without validating it with `save(validate: false)`
4.  change model validations without updating the data in the database
5.  concurrent data updates bypassing unique constraints
    Worker to verify data integrity

```ruby
class ModelDataIntegrityCheckerJob
  include Sidekiq::Worker
  sidekiq_options retry: false

  def perform(klass_name)
    klass = klass_name.constantize
    invalid_objects = []

    klass.find_each do |object|
      unless object.valid?
        invalid_objects << [object.id, object.errors.full_messages]
      end
    end

    if invalid_objects.present?
      raise "Invalid state for #{klass} objects: #{invalid_objects}"
    end

    invalid_objects
  end
end

ModelDataIntegrityCheckerJob.perform_async(User.name)
```

## not null constraints

```ruby
class AddNotNullConstraints < ActiveRecord::Migration[5.2]
  def change
    change_column_default :users, :boolean_field, false
    change_column_null :users, :boolean_field, false
  end
end
```

## deleting children in rails children

```ruby
class AddDeleteChildren < ActiveRecord::Migration[5.2]
  def change
    add_foreign_key :children, :parents, on_delete: :cascade
  end
end
```
