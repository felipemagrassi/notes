# Commands - 2023-05-21 17:59:12.004205558 -0300 -03 m=+0.051198059

tags: #arkency #fleeting #commands

Commands are structures that represent the attributes necessary to perform an operation in your system. Usually
they can be represented as a `ActiveSupport::HashWithIndifferentAccess` or a `Struct` with the necessary attributes.
Commands can perform simple validations, like checking if a field is present or if a field is valid.

E.g.:

### Type-casting with active\_model

```ruby

require 'active_model'

module Command
  ValidationError = Class.new(StandardError)

  def self.included(base)
    base.include ActiveModel::Model
    base.include ActiveModel::Validations
    base.include ActiveModel::Conversion
  end

  def validate!
    raise ValidationError, errors unless valid?
  end
end
```

```ruby
class ExpireOrderCommand
  include Command

  attr_reader :order_number
  validates_presence_of :order_number

  def order_number=(int)
    @order_number = Integer(int)
  end
end

```

### Virtus.model

```ruby
class LikeProductsCommand
  include Command
  include Virtus.model

  attribute :product_ids, [Integer]
  attribute :value,       Boolean

  validates :product_ids, presence: true
end

```

```ruby
class OrderLine
  include Virtus.model
  attribute :quantity, Integer
  attribute :product_type_id, Integer
  attribute :price, BigDecimal
  attribute :seats, [Seat]
end

class DeliveryAddress
  include Virtus.model
  include ActiveModel::Validations

  attribute :first_name,   String
  attribute :last_name,    String
  attribute :street_line,  String
  attribute :street_line2, String
  attribute :zip_code,     String
  attribute :city_name,    String
  attribute :country_code, String

  validates :first_name,
            :last_name,
            :street_line,
            :zip_code,
            :city_name,
            :country_code,
            presence: true
end

class Input
  include Virtus.model
  include ActiveModel::Validations

  attribute :payment_method,   String
  attribute :delivery_address, DeliveryAddress
  attribute :order_lines,      [OrderLine]
end

class ChangePricingCommand
  include Virtus.value_object
  values do
    attribute :product_id, Integer
    attribute :price,    GrossAmount
    attribute :kickback, BigDecimal, default: 0.0
  end
end

```

### Dry::Struct

```ruby
module Types
  include Dry::Types.module

  ComicPreference = Strict::String.enum("Marvel", "DC")
end

class RegisterReader < Dry::Struct
  attribute :name, Types::Strict::String
  attribute :age, Types::Strict::Int
  attribute :preference, Types::ComicPreference
end

```

## Creating a command bus within a service

```ruby

require 'arkency/command_bus'

class OrdersService
  def initialize(event_store:)
    @repository = AggretateRoot::Repository.new(event_store)

    @command_bus = Arkency::CommandBus.new
    { SubmitOrderCommand  => method(:submit),
      ExpireOrderCommand  => method(:expire),
      CancelOrderCommand  => method(:cancel),
      ShipOrderCommand    => method(:ship),
    }.map{|klass, handler| @command_bus.register(klass, handler)}
  end

  def call(*commands)
    commands.each do |cmd|
      @command_bus.call(cmd)
    end
  end

  private

  def submit(cmd)
    # ...
  end

  def expire(cmd)
    cmd.validate!
    @repository.with_aggregate(
      Order.new(number: cmd.order_number),
      "Order$#{cmd.order_number}"
    ) do |order|
      order.expire
    end
  end

  def cancel(cmd)
    # ...
  end

  def ship(cmd)
    # ...
  end
end

```

[[domain-driven-rails]]

