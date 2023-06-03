# Bounded Contexts - 2023-06-03 10:26:22.76202836 -0300 -03 m=+256.810369776

Bounded Contexts are a way to divide a large domain into smaller parts.
Making it easier to introduce a new developer into a years-old project,
or to introduce a new feature into a large project. Making it easier to
divide and conquer, breaking problems into smaller chunks

## Tactical vs Strategic patterns

Tactical focus on 'items' used to build the model. Strategic
focus on concepts, on communication and definition of ubiquitous language

As projects grow, they tend to become a Big Ball of Mud. Bounded Contexts
are a way to avoid that. Different groups of people will use different
vocabularies in different parts of a large organization. DDD divides up
a large system into Bounded Contexts, each having a unified model.

## Creating smaller models and clear data ownership per departments

Imagine a system that have many perspectives on the same data, maybe, forecasting,
pricing, inventory, sales, etc. Each of these departments will have a different
perspective on the same data and different needs. DDD creates smaller models
to clear the data ownership per department. Having a Sales::Product, Pricing::Product
Invetory::Product, with their own attributes and methods, and a shared
Product::Product that will be used by all of them. Meaning that a quantity
attribute will be used by Inventory::Product, but not by Sales::Product.
A price attribute can only be changed by Pricing::Product, but not by
Inventory::Product.


