# Debugging rails itself - 2023-05-15 00:42:08.066379912 -0300 -03 m=+3504.898151876

tags: #rails #debugging #talks 

## What are bugs

The term originates from the days of vacuum tubes computers
where a bug would fly into the tubes and cause a short circuit.
But we can also think of bugs as a difference between the intended
behavior and the observed behavior.

A intended behavior is not always a behavior you would expect. This
type of thinking violates the principle of least astonishment, where
people are surprised by the behavior of a system.

Bugs usually will crop out when the system is under stress, when
the system is used in a way that was not anticipated, or when
some interaction between two or more components of the system
is not well tested.

## Creating a minimal testing environment

### Active Record minimal testing environment

```ruby

# Inline gemfile
gemfile(true) do
    source 'https://rubygems.org'
    git_source(:github) { |repo| "https://github.com/#{repo}.git" }

    gem 'rails', github: 'rails/rails'
    gem 'sqlite3'
end

ActiveRecord::Base.establish_connection(adapter: 'sqlite3', database: ':memory:')
ActiveRecord::Base.logger = Logger.new(STDOUT)

ActiveRecord::Schema.define do
    create_table :posts, force: true do |t|
    end

    create_table :comments, force: true do |t|
        t.integer :post_id
    end
end

class BugTest < Minitest::Test
    def test_association_stuff
        post = Post.create!
        post.comments << Comment.create!

        assert_equal 1, post.comments.count
        assert_equal 1, Comment.count
        asset_equal post.id, Comment.first.post.id
    end
end

```

## Tools

### Git Bisect

Helps you find the commit that introduced a bug.

```bash
git checkout master
git bisect start
git bisect bad
git checkout v5.2.0
git bisect good
```
