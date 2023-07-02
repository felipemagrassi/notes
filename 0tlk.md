# Aggregates - 2023-05-18 21:30:33.003747084 -0300 -03 m=+0.013695721

---

tags: #arkency #domain-driven-rails

Aggregates are a collection of objects that are bound together by a root entity,
otherwise known as the aggregate root.

The role of an aggregate is to protect business rules, to protect from invalid state.

For an Order aggregate, is making sure that discounts are properly applied and calculated, taxes are added, and so on.
A collection of business requirements are coherent and calculated together at the end of a transaction.

Sometimes will be just one object, one entity, sometimes will be a collection of entities.

## Why is important to have aggregates in rails?

In Rails, we can naturally grow an application and start with a single model, and then add more models, and more models, and more models.

This creates a ever-growing cluster of objects and models that are not really connected to each other.
Allowing the application to to grow and create commands/operations/actions in a way that gets more and more complex.

To test something it becomes harder, having the need of a lot of dependencies, objects become more coupled to each other and
it becomes harder to control where things are happening.

This makes rails suffer from a "brain split", where some operations, some business rules are happening in memory,
but some other business rules are happening in the database.

## Problem with big aggregates

Big aggregates means big changes in a single transaction, which means that we are locking a lot of data in the database,
which means that we are not able to scale the application. We need optimistic and pessimistic locking to make sure that
we are not having concurrency issues.

So if a aggregate is too big, let's say that we have a big order, and we have a lot of items in the order,
you would need to

1.  get an aggregate state from DB and pessimistically lock it
2.  perform the necessary operations in-memory on the objects
3.  serialize the state to DB from memory
4.  release the lock

If this takes about 0.2s per transaction, and we have 1000 transactions per second, we would need 200s to process all of them.
This is not feasible in a real world where 1000 customers are buying things at the same time.

So when thinking about how big aggregates should be, we need to think some things:

- What business rules it needs to protect
  - Which of them need to be protected in a single transaction
  - Which of them could be delayed a few seconds with eventual consistency
- How often is it going to be changed in the worst case ( 1000 times per second? 100 times per second? 10 times per second? )
- How many separate users will be changing it at the same time?

## Updating aggregates

A good rule of thumb is to update one aggregate in one transaction. But this is not always possible, there are cases where
is simpler to update multiple aggregates in a single transaction and it makes sense. But there are cases where
these rules are better to be followed religiously.

E.g. Having a webhook of a successful payment, and we need to update the order and the user, and we need to send an email to the user.
We could do all of this in a single transaction, but it would be better to have a background job to do this, and have eventual consistency.

Another thing to keep in mind is coupling, when updating multiple aggregates in a single transaction, we are coupling them together,
possibly making the harder to refactor, make sure that the transaction doesn't take too long and the are updated by the same user.

## Conceptualizing aggregates

Imagine you have a ticketing system, where you have tickets for multiple events, soccer matches, concerts, conferences... The booking
process is very complicated, there is the need of making sure that the user is not buying more tickets than allowed, that the user
is not buying tickets already bought and so on.

You also need to update the sales stats, the number of tickets sold, the number of tickets available, the number of tickets sold by
some event, the number of tickets sold by some event in some category, and so on.

One way to implement it, it using one database transaction

- open transaction
- place order
- update Venue object and check business rules
- change Inventory object and check business rules
- update Balance
- do some other stuff
- commit

In most scenarios, this is ok and works. But there is a small potential problem here. Saying that the transaction
takes 0.2s, and it takes about to 0.6s to finish all the operations, this could be painful in some cases,
but it's still acceptable. With these contention points, we can figure out how many transactions per second we can handle.

Handling a transaction in 0.6s we can assure, we can assure 1-2 transactions per second. It's ok in a lot of cases, but
when handling a Adele concert, we are having some problems.

In the approach of having one single DB transaction, we can just rollback everything and it's ok.

But there is another approach, where we have multiple smaller transactions, we just can't rollback everything, we need
domain event handlers, process managers, definitely having more manual work and complexity

## ActiveRecord Aggregates

One problem with aggregates in ActiveRecord is that, the objective of aggregates is
to protect business rules, and everything in ActiveRecord is public.

e.g.

```ruby
class Basket < ActiveRecord::Base
  has_many :items, autosave: true

  # Rules:
  #
  # - items with the same SKU are not repeated
  # but instead their quantity is increased
  #
  # - basket total quantity is sum of items’ qty
  def add_item(sku, quantity)
    changed_item   = items.find{|oi| oi.sku == sku }
    changed_item ||= items.build(
      sku: sku,
      quantity: 0
    )
    changed_item.quantity += quantity
    self.total_quantity += quantity
  end
end

Basket.transaction do
  # get the whole aggregate (cluster of objects) from DB
  # using pessimistic locking to guarantee no conflicts
  basket = Basket.preload(:items).lock.find(1)

  # perform a business operation
  basket.add_item("BOOK1", 2)

  # persist changes back
  basket.save!
end
```

A new developer in the team could easily do this:

```ruby
basket_item = Item.find(23) # same problem as before
basket_item.quantity += 1
basket_item.save!
```

Editing the item directly, and not using the aggregate root. This is a problem, because we are not protecting the business rules
of the aggregate, so using ActiveRecord requires a lot of discipline to protect the business rules from the team.

[[domain-driven-rails]]
