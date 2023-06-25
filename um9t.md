# Value Objects - 2023-06-03 10:26:47.977239326 -0300 -03 m=+282.025580752

Value Objects are objects with no identify.
Since they have no identity, comparing them is possible

They are designed to be immutable and if something changes, a new object is created.
You should not be able to an attribute inside a value object.

The benefit of using Value Objects is that you can pass them around, 
without passing from method to method or class to class.
And they can protect simple business rules

[[npdb]]

## Value Objects characteristics

*   Immutable
*   No identity
*   Values comparable
*   Compound attributes


[[domain-driven-rails]]

