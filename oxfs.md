# Domain Events - 2023-05-21 20:11:55.863460179 -0300 -03 m=+7963.910452690

tags: #arkency

Domain Events are structures to describe what happened in your app.

## Anatomy of a Domain Event

First, what not to do:

1.  Naming

Can be misleading when doesn't describe exactly the business change.
Some possible examples may be:

*   Something that has already happened, named in past tense and with ubiquitous language
*   Represents a state change
*   Something that will never change

E.g. `ProductItemSold` because can me misleading, when having a ticketing system, we sell tickets, so having
a `TicketSold` event would be more appropriate.

2.  Stop with the CRUD

Try to describe better the business change, instead of using CRUD terms.
A user is never created, it's registered, signed up, imported, etc.

So creating a `UserCreated` event is not ideal, it's better to use a `UserImportedByAdmin`,
`UserSignedUp`, `UserRegistered`, etc.

3.  Your event is not your entity

An event is not an entity, it's a representation of a business change that happened to an entity.
Having an event means having only the data needed to describe the change, not the whole entity.

The only necessary data, is the ubiquious language of the change. E.g. `TicketSold` event
only needs the barcode that represents the sale.

Having barcode as the key event, is better because is a natural part of the solution.
E.g. Sales - token

4.  Modelling events require temporal thinking

## Should we save domain events?

Yes, we should save domain events, because they are the source of truth of what happened in our system.
For auditing, debugging, etc.

And saving as a pub/sub mechanism, we can have other systems listening to those events and do something
with them.

## Schema

We can use a schema to validate the data of the event, so we can be sure that the data is correct.

```ruby

class PaymentNeedsMoreTime < RailsEventStore::Event
  SCHEMA = {
    order_id:             Integer,
    payment_id:           Integer,
    payment_gateway_name: String,
    seconds_needed:       Integer,
  }

  def self.strict(data, **attr)
    ClassyHash.validate(data, SCHEMA, strict: true, verbose: true)
    new(data, **attr)
  end
end

stream_name = "payment_#{payment.id}"
event = PaymentNeedsMoreTime.strict(
  order_id: payment.order_id,
  payment_id: payment.id,
  payment_gateway_name: "v8",
  seconds_needed: 30*60*60,
)
client.publish(event, stream_name)
```

## Splitting big events into smaller ones

Sometimes we have a big event, that has a lot of data, and we want to split it into smaller events.

E.g. In a ticketing system, we can have a barcode being scanned and decide if the ticket is valid or not.

If we have a device that does all the process of having the barcode scanned, validating the attributes
and opening the venue, we can have a big event like `TicketScanned` with all the data.

But this couples the design of the application, because if we change the device and the barcode
is responsibility of another system, we would be in a better situation having `BarcodeScanned`,
`RulesValidatedPositively`, `EnteredVenue`.

E.g.

When we have a invoice asked for, only one command is sent to the server, but multiple things can happen
in the server, we could schedule some jobs, send emails, etc.

So we could have a `InvoiceRequested`, `InvoiceGenerated`, `CreditNoteDeliveryScheduled`, `InvoicePaymentReminderScheduled`
events.

## Domain Events on failure

So here are some examples of domain events, which are all about failures:

`StartRemoteTransactionFailed`

Some payment gateways that we work with require an API request to create the transaction on their side first. This is done synchronously and can be very problematic when not working correctly. It's good to know and be able to react to such failures.

`PaymentFailed`

A bit different than the previous one. The technical infrastructure worked but the client probably did not have enough money. But it could be also that there were problems with 3D-Secure or payment authorization.

`InvoiceDeliveryFailed` / `InvoiceDeliveryRescheduled`

Obviously, we want our customer to pay their invoices. But it is hard to expect it if delivering the email containing it fails. In such case, it is good to know when it was rescheduled to.

`ErrorProcessingBigBankFile`

There is a BigBank™, which sends us improperly formatted CSV files (I know, facepalm...) that we download and process daily. Most of the time the file is good enough to automatically unmangle it. But sometimes it is messed up beyond our code capabilities. This requires developer attention, business and customer support awareness as it delays some automatic processes that happen on the platform. Sounds important enough to deserve a domain event.

SalesforceEvents

`AccountCreateFailed`
`OpportunityUpdateFailed`
`OpportunityOwnerNotFound`
In one project our integration with Salesforce can be sometimes be a bit ... fragile. The way they handle permissions for APIs is still a bit of mystery for me. We have domain events published when the integration is not working correctly with the errors included. Based on those cryptic errors sales people can change Salesforce data and fix the integration without a developer's help.

`OrderCannotBeCompleted`

Even when a payment is successful, when we try to complete the order and ship all the related virtual goods, errors can still occur (due to bugs or more likely final business rules being violated). It's a very important domain event because we have the customer's money but we can't ship the products. Later you will learn how you can compensate in such situations.

`OrderEmailDeliveryFailed`

Even when you got paid, completed all the virtual products you want to deliver, the customer might still not get them because sending an email can fail.

`Postal::UploadFailed`

Similar to the last one, but related to the integration with postal API. When a customer wants tickets printed and shipped by a real postman, it is important to know and react when the integration fails.

## Where should domain events be published?

Domain events should be published in the same transaction as the command that caused them.

### Model

```ruby
class Basket < ActiveRecord::Base
  has_many :items, autosave: true

  def add_item(sku, quantity)
    changed_item   = items.find{|oi| oi.sku == sku }
    changed_item ||= items.build(
      sku: sku,
      quantity: 0
    )
    changed_item.quantity += quantity
    self.total_quantity += quantity
    event_store.publish(ItemAddedToBasket.strict(data: {
      basket_id: id,
      sku: sku,
      quantity: quantity
    }), stream_name: "Basket$#{id}")
  end

  private

  def event_store
    Rails.configuration.event_store
  end
end
```

### Model + Callback

```ruby
class Basket < ActiveRecord::Base
  has_many :items, autosave: true

  def unpublished_events
    @unpublished_events ||= []
  end

  after_save do
    unpublished_events.each do |ev|
      event_store.publish(ev, stream_name: "Basket$#{id}")
    end
  end

  def add_item(sku, quantity)
    changed_item   = items.find{|oi| oi.sku == sku }
    changed_item ||= items.build(
      sku: sku,
      quantity: 0
    )
    changed_item.quantity += quantity
    self.total_quantity += quantity

    self.unpublished_events << ItemAddedToBasket.strict(
      basket_id: id,
      sku: sku,
      quantity: quantity
    )
  end

  private

  def event_store
    Rails.configuration.event_store
  end
end
```

### Service

```ruby
class BasketsService
  def initialize(event_store)
    @event_store = event_store
  end

  def add_item(basket_id:, sku:, quantity:)
    ActiveRecord::Base.transaction do
      basket = Basket.lock.find(basket_id)
      item = basket.add_item(sku, quantity)
      basket.save!
      @event_store.publish(ItemAddedToBasket.new(data: {
        basket_id: b.id,
        sku: item.sku,
        quantity: item.quantity}),

        stream_name: "Basket$#{basket.id}"
      )
    end
  end
end
```

## Event Streams

Event streams are a way to group events together. They are identified by a stream name.
Streams helps querying events, and also helps with concurrency.

*   by time
    *   daily
    *   hourly
    *   monthly
*   geographical
    *   by country
    *   by city
    *   by region
*   user based
    *   by user
    *   by user and type
*   long running processes
    *   by process id
    *   by process id and type

```ruby
class LinkToDailyStream
  def initialize(event_store = Rails.configuration.event_store)
    @event_store = event_store
  end

  def call(event)
    stream = "day-#{event.timestamp.strftime('%F')}"
    @event_store.link(event.event_id, stream_name: stream)
  end
end

Rails.configuration.event_store.subscribe_to_all_events(LinkToDailyStream.new)
```

[[domain-driven-rails]]

