# Rspec testing assertions - 2023-05-15 00:25:12.665704323 -0300 -03 m=+2489.497476287

In this post we will see how to use the `match_array` and `contain_exactly` assertions in #rspec.

```ruby
require 'rspec'
require 'rspec/autorun'

Person = Struct.new(:name, :age, :gender, :alive, :email, :phone, :favorite_color)

RSpec.describe 'easier testing assertions' do
  let(:array) do 
    [
      Person.new('Leon', 18, 'male', true, 'leon@email.com', '555-555-5555', 'red'),
      Person.new('Jane', 20, 'female', true, 'jane@email.com', '555-555-5555', 'blue')
    ]
  end

  describe 'hard to assert equality' do
    result = [Person.new('John', 18, 'male', true, 'leon@email.com', '555-555-5555', 'red')]

    it 'returns people with a favorite color of red' do
      expect(array).to eq(result)
    end
  end  

  describe 'easy to assert equality' do 
    result = [Person.new('John', 18, 'male', true, 'leon@email.com', '555-555-5555', 'red')]

    it 'returns people with a favorite color of red' do
      expect(array.map(&:favorite_color)).to eq(result.map(&:favorite_color))
    end
  end

end
```

*Results:*

```
FF

Failures:

  1) easier testing assertions hard to assert equality returns people with a favorite color of red
     Failure/Error: expect(array).to eq(result)

       expected: [#<struct Person name="John", age=18, gender="male", alive=true, email="leon@email.com", phone="555-555-5555", favorite_color="red">]
            got: [#<struct Person name="Leon", age=18, gender="male", alive=true, email="leon@email.com", phone="555-5..., gender="female", alive=true, email="jane@email.com", phone="555-555-5555", favorite_color="blue">]

       (compared using ==)

       Diff:
       @@ -1,2 +1,3 @@
       -["#<struct Person name=\"John\", age=18, gender=\"male\", alive=true, email=\"leon@email.com\", phone=\"555-555-5555\", favorite_color=\"red\">"]
       +["#<struct Person name=\"Leon\", age=18, gender=\"male\", alive=true, email=\"leon@email.com\", phone=\"555-555-5555\", favorite_color=\"red\">",
       + "#<struct Person name=\"Jane\", age=20, gender=\"female\", alive=true, email=\"jane@email.com\", phone=\"555-555-5555\", favorite_color=\"blue\">"]
     # /tmp/mdeval//easiertestingassertionsmd_1_32.rb:18:in `block (3 levels) in <main>'

  2) easier testing assertions easy to assert equality returns people with a favorite color of red
     Failure/Error: expect(array.map(&:favorite_color)).to eq(result.map(&:favorite_color))

       expected: ["red"]
            got: ["red", "blue"]

       (compared using ==)
     # /tmp/mdeval//easiertestingassertionsmd_1_32.rb:26:in `block (3 levels) in <main>'

Finished in 0.01029 seconds (files took 0.03815 seconds to load)
2 examples, 2 failures

Failed examples:

rspec /tmp/mdeval//easiertestingassertionsmd_1_32.rb:17 # easier testing assertions hard to assert equality returns people with a favorite color of red
rspec /tmp/mdeval//easiertestingassertionsmd_1_32.rb:25 # easier testing assertions easy to assert equality returns people with a favorite color of red

```

```ruby
User = Struct.new(:name, :type, :since)

class UserRepo
  def initialize(user_list)
    @user_list = user_list
  end

  def eligible_for_treats(now = Time.now)
    @user_list
  end
end

require "rspec/autorun"

RSpec.describe UserRepo do
  it "identifies users eligible for special treats" do
    u1 = User.new("Frankie Freeloader", :free, Time.new(2014, 1, 1))
    u2 = User.new("Nelly Newbie", :premium, Time.new(2014, 11, 15))
    u3 = User.new("Larry Longtimer", :premium, Time.new(2014, 1, 1))
    u4 = User.new("Suzie Special", :vip, Time.new(2014, 11, 15))

    repo = UserRepo.new([u1, u2, u3, u4])
    expect(repo.eligible_for_treats(Time.new(2014, 11, 30)).map(&:name)).
      to eq(["Larry Longtimer", "Suzie Special"])
  end
end
```

*Results:*

```
F

Failures:

  1) UserRepo identifies users eligible for special treats
     Failure/Error:
       expect(repo.eligible_for_treats(Time.new(2014, 11, 30)).map(&:name)).
         to eq(["Larry Longtimer", "Suzie Special"])

       expected: ["Larry Longtimer", "Suzie Special"]
            got: ["Frankie Freeloader", "Nelly Newbie", "Larry Longtimer", "Suzie Special"]

       (compared using ==)
     # /tmp/mdeval//easiertestingassertionsmd_76_103.rb:23:in `block (2 levels) in <main>'

Finished in 0.00864 seconds (files took 0.03885 seconds to load)
1 example, 1 failure

Failed examples:

rspec /tmp/mdeval//easiertestingassertionsmd_76_103.rb:16 # UserRepo identifies users eligible for special treats

```
