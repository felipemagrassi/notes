# Event Handler - 2023-05-22 20:34:37.59902047 -0300 -03 m=+36.749368478

tags: #arkency #event-handler

Event handlers are ways for decoupling code and maintaining boundaries. We can react to something
that happened in our boundaries or in other boundaries.

E.g.

### Order reacting to Payment being paid

After a few projects which involved handling transactions and money there is one thing I learned for sure.
Getting the money and handling orders, on the one hand, and handling their delivery on the other, are completely different processes with different requirements.
You can get money for orders that expired or were canceled. You can get the money but not be able to deliver the product.
There are thousands of reasons to keep Payments as a separate context in your app.
Usually, the complexity of reacting to payment is much greater than just marking payment as paid.
In all the projects that I have worked on, the logic of the Payment being paid itself was minimal: just an update here or there, nothing big.
But the logic of the whole system that now needs to react to it? 10 times bigger.
It does not matter if it was about granting access to movies, delivering tickets or selling pillows.
The business logic is more complex, more prone to failure in other parts of the system.
And you can't risk a rollback of the information that the payment was successful.
The obvious best solution for those situations is to keep the logic separate. Payment was paid, cool. Now let the rest of the system react to it.

### Scanner (when refunded -> revoke access)

In one project, a ticketing application, we have a Scanner bounded context. It might sound strange, because how complicated could it be to scan a barcode?
Remember that it was `scanned_at`, and that's it, right? That's what we thought as well, but it turned out to be a more complex beast.
You learn that you need to support leaving the venue (for a smoke or breath of fresh air) and coming back.
There are also multi-day events, when every day you may return and need to be scanned again.
There are venues with multiple gates and sometimes you must use a certain gate to enter the venue,
e.g., in a stadium you might be required to use the gate closest to your seat.
So technically you may enter, but not from this side of the stadium.
And there are venues with poor Internet connections that need to support offline scanning and synchronization in the background.
When you combine all of that, you have a bounded context :)
In this system, when a ticket/order gets refunded, the scanner context reacts to it and revokes access for certain barcodes.

### Unbook seats when placing order failed

In one part of the code, we react to an error when placing an order by unbooking seats that were previously reserved for it.

### Pushing to segment.io, getvero, salesforce and other data sinks

In the world of multiple services available via API which help us build better software,
oftentimes when something important happens we need to notify them with some payload.

### Sending metrics when payment paid

It's also nice to send metrics (i.e. to chillout) so admins, CEOs, CTOs and others can see what's going on in our
system and so DevOps can react in case of an emergency.

### mobile/sms notifications

### sending emails when user imported

### read models

In sum, when we hear about event handlers, our first thought should be about SIDE-EFFECTS.

## Services and side effects

In the world of microservices, we have a lot of services that are responsible for doing one thing and doing it well.
We have a service that is responsible for handling payments, another one for sending emails, another one for sending SMS messages, etc.

When introducing services to ruby, we can have a service with many dependencies, but it is important to remember that
sometimes we do not have real dependencies to use the service.

```ruby
class UserService
  def initialize(a, b, c, d)
    @a, @b, @c, @d = a, b, c, d
  end

  def register
    register_user

    @a.user_registered(user)
    @b.notify_about_user(user.id)
    @c.greet_user(user.email)
    @d.new_user_happened(user.login)
  end
end
```

**Many parts of our system need to know that something happened, but they don't participate in that thing happening.**

## What if A, B, C, or D fails? Should we roll the transaction back?

If so, then you might have better consistency (assuming the dependencies do not contact other databases or APIs). But it might also mean that a small problem or bug, in a less-important part of your system prevents the user from registering.

Often it might be better if the user registers, A, B, C react properly and are in a consistent state but when D fails, only D remains affected and there is small inconsistency in our system in one of its parts.
Developers can get notified with an exception tracker about the problem in D and fix it manually or automatically (maybe via retrying) without blocking the (more important) user registration.

## Do we want to wait for A, B, C, D? Do we need to?

All of those things take time. Perhaps it's better to show the registration result faster to the customer.

## Using `after_commit` hooks

Callbacks are just bad. They are hard to test, they are hard to reason about, they are hard to debug.
But sometimes they are our only option. If we want to react to something that happened in our system, we need to use them.

Example for `ElasticSearch` indexing

```ruby
class Order < ActiveRecord::Base
  after_commit do |order|
    Elasticsearch::Model.client.index(
      id: id,
      body: {
        id:              id.to_s,
        shop_id:         shop_id,
        buyer_name:      buyer_name,
        email:           buyer_email,
        state:           state,
        created_at:      created_at
    })
  end
end
```

### What if we want to react to something that happened in another system?

What I recommend, instead of using Active Record callbacks,
is publishing domain events such as `UserRegisteredViaEmail`, `UserJoinedFromFacebook`, `UserImported`, `OrderPaid` and so on

## Cut heavy event handlers in testing

When we have a lot of event handlers, we might want to cut them in tests.
`PdfGenerating`, `EmailSending`, `SmsSending` and so on.

## Sync Event Handlers

Sync event handlers are the simplest ones. They are just methods that react to something that happened in our system.
They give us decoupling in space, but not in time. They are synchronous, so they block the execution of the code that triggered them.

### Example

Using `rails-event-store`

```ruby
Rails.configuration.event_store.tap do |es|
  es.subscribe(
    ->(event) {
      OrderList::OrderSubmittedHandler.new.call(event)
    }, to: [Orders::OrderSubmitted]
  )
end

module OrderList
  class OrderSubmittedHandler
    def call(ev)
      # code ...
    rescue => ev
      ErrorNotifier.notify(ev)
    end
  end
end
```

As the sync event handler executes in the same open db transaction, it can rollback the transaction if something goes wrong.

## Async Event Handlers

Async handlers give us decoupling in time. They are asynchronous, so they do not block the execution of the code that triggered them.
The handler is executed in a separate process, thread, or even on a different machine.

To use async handlers, we need infrastructure to support it. We need a queue, a worker, and a way to communicate between them.
Examples of such infrastructure are Sidekiq, Resque, RabbitMQ, Kafka, and so on.

### Example

Using `rails-event-store`

```ruby
Rails.configuration.event_store.tap do |es|
  es.subscribe(
    ->(event){
      Discounts::Process.perform_later( YAML.dump(event) )
    }, to: [Orders::OrderShipped]
  )
end
```

## Handling rollbacks and crashes

```ruby
Rails.configuration.event_store.tap do |es|
  es.subscribe(
    ->(event){
      WelcomeInSystem.perform_later( YAML.dump(event) )
    }, to: [UserRegisteredFromEmail]
  )
  es.subscribe(
    ->(event){
      GrantFreeCredits.perform_later( YAML.dump(event) )
    }, to: [UserRegisteredFromEmail, UserJoinedFromFacebook]
  )
end

```

```ruby
ActiveRecord::Base.transaction do
  User.create!(...)
  event_store.publish( UserRegisteredFromEmail.strict(...) )

  # ...

  raise ActiveRecord::Rollback
  # or server crash
end

```

In case of a SQL based database, we can use a transaction to rollback the changes in the database.

*   User creation is rolled back
*   stored domains are rolled back
*   `WelcomeInSystem` and `GrantFreeCredits` are not executed

In case of a NO-SQL database, like a Redis + Sidekiq

```ruby
Rails.configuration.event_store.tap do |es|
  es.subscribe(
    ->(event){
      WelcomeInSystem.perform_later( YAML.dump(event) )
    }, to: [UserRegisteredFromEmail]
  )
  es.subscribe(
    ->(event){
      GrantFreeCredits.perform_later( YAML.dump(event) )
    }, to: [UserRegisteredFromEmail, UserJoinedFromFacebook]
  )
end
```

```ruby
ActiveRecord::Base.transaction do
  User.create!(...)
  event_store.publish( UserRegisteredFromEmail.strict(...) )

  # ...

  raise ActiveRecord::Rollback
  # or server crash
end
```

*   user creation (SQL) is rolled back
*   stored domain events are rolled back (in the default case of the rails\_event\_store gem and using active record for storing those events)
*   `WelcomeInSystem` and `GrantFreeCredits` handlers (which are just ActiveJob background jobs saved in Redis) are not rolled back.

How can we solve this? Executing the scheduling after commit

```ruby
class AfterCommitJob
  def initialize(handler, payload)
    @handler = handler
    @payload    = payload
  end

  def has_transactional_callbacks?
    true
  end

  def before_committed!
  end

  def committed!(*_, **__)
    @handler.perform_later(@payload)
  end

  def rolledback!(*_, **__)
    logger.warn("Transaction rolledback! - async handlers skipped")
  end

  def logger
    Rails.logger
  end
end
```

```ruby

Rails.configuration.event_store.tap do |es|
  es.subscribe(
    ->(event){
      serialized_event = YAML.dump(event)
      if ApplicationRecord.connection.transaction_open?
        ApplicationRecord.connection.current_transaction.add_record(
          AfterCommitJob.new(WelcomeInSystem, serialized_event)
        )
      else
        WelcomeInSystem.perform_later(serialized_event)
      end
    }, to: [UserRegisteredFromEmail]
  )
end
```

This makes sure that the handler is executed only if the transaction is committed.

It can fails when:

*   The application server crashes after it committed data to the SQL DB but before it scheduled the job in Redis
*   The SQL DB is working but Redis is unavailable. This can be worked around by saving the job to the SQL DB

### Examples

1.  AfterCommitAsyncDispatcher

```ruby
event_store = RailsEventStore::Client.new(
  dispatcher: RubyEventStore::ComposedDispatcher.new(
    RailsEventStore::AfterCommitAsyncDispatcher.new(scheduler: RailsEventStore::ActiveJobScheduler.new),
    RubyEventStore::Dispatcher.new
  )
)
```

2.  Implement you event handlers as async ones

```ruby
class WelcomeInSystem < ActiveJob::Base
  prepend RailsEventStore::AsyncHandler

  def perform(event)
    # ... do the work here
  end
end
```

3.  Subscribe event handler to domain events

```ruby
Rails.configuration.event_store.tap do |es|
  es.subscribe(WelcomeInSystem, to: [UserRegisteredFromEmail])
end
```

## SQL to NO-SQL

Besides the already mentioned solution of using only an SQL transactional DB there is one that I know about: save everything in SQL
and have a separate (monitored) process move the data from SQL to external message queues such as Redis, Kafka, RabbitMQ with retries and proper idempotency.

https://blog.arkency.com/2017/04/reliable-notifications-between-two-systems/

## Eventual Consistency

The more we try to do in a single transaction, the more likely it is that we will have to deal with rollbacks and crashes.
Big Transactions can cause problems and failures even from something as simple as bugs in our code, leading to deadlocks and contentions

When splitting our app into multiple aggregates we need to decide:

*   Size of the aggregate, data it keeps, business rules enforced synchronously
*   rules can be guaranteed by using eventual consistency

We don't want our transactions to be coupled between multiple bounded contexts.
Because we want to leave an option to decouple them later into separates microservices, if
they grow too big.

Some processes can be asynchronous, and we can use eventual consistency to make them work.

## Ticketing Importing Example

```ruby
class Season::PassImported < RubyEventStore::Event
  SCHEMA = {
    id: Integer,
    barcode: String,
    first_name: String,
    last_name: String,
    email: String,
  }.freeze

  def self.strict(data:)
    ClassyHash.validate(data, SCHEMA, strict: true, verbose: true)
    new(data: data)
  end
end

Rails.configuration.event_store.tap do |es|
  es.subscribe(->(event) do
    IdentityAndAccess::RegisterSeasonPassHolder.perform_later(YAML.dump(event))
  end, to: [Season::PassImported])

  es.subscribe(->(event) do
    Season::AssignUserIdToHolder.perform_later(YAML.dump(event))
  end, to: [IdentityAndAccess::UserImported, IdentityAndAccess::UserAlreadyRegistered])
end

ActiveRecord::Base.transaction do
  pass = Season::Pass.create!(...)
  event_store.publish(Season::PassImported.strict(data: {
    id: pass.id,
    barcode: pass.barcode,
    first_name: pass.holder.first_name,
    last_name: pass.holder.last_name,
    email: pass.holder.email,
  }), stream_name: "pass$#{pass.id}")
end

class IdentityAndAccess::UserImported < RubyEventStore::Event
  SCHEMA = {
    id: Integer,
    email: String,
  }.freeze

  def self.strict(data:)
    ClassyHash.validate(data, SCHEMA, strict: true, verbose: true)
    new(data: data)
  end
end

class IdentityAndAccess::UserAlreadyRegistered < RubyEventStore::Event
  SCHEMA = {
    id: Integer,
    email: String,
  }.freeze

  def self.strict(data:)
    ClassyHash.validate(data, SCHEMA, strict: true, verbose: true)
    new(data: data)
  end
end

module IdentityAndAccess
  class RegisterSeasonPassHolder < ApplicationJob
    prepend RailsEventStore::AsyncHandler
    queue_as :default

    def initialize(event_store = Rails.configuration.event_store)
      @event_store = event_store
    end

    def perform(event)
      ActiveRecord::Base.transaction do
        user = User.create!(email: event.data.email)
        @event_store.publish(UserImported.strict(data: {
          id: user.id,
          email: user.email,
        }), stream_name: "user$#{user.id}")
      end
    rescue User::EmailTaken => exception
      @event_store.publish(UserAlreadyRegistered.strict(data: {
        id: exception.user_id,
        email: exception.email,
      }), stream_name: "user$#{exception.user_id}")
    end
  end
end

module Season
  class AssignUserIdToHolder < ApplicationJob
    prepend RailsEventStore::AsyncHandler
    queue_as :default

    def initialize(event_store = Rails.configuration.event_store)
      @event_store = event_store
    end

    def perform(event)
      ActiveRecord::Base.transaction do
        Pass.all_with_holder_email!(event.data.email).each do |pass|
          pass.set_holder_user_id(event.data.id)
          @event_store.publish(Season::PassHolderUserAssigned.strict(data: {
            pass_id: pass.id,
            user_id: pass.holder.user_id,
          }), stream_name: "pass$#{pass.id}")
        end
      end
    end

  end
end
```
