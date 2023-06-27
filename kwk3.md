# Interface Segregation Principle - 2023-06-26 22:48:50.232380783 -0300 -03 m=+15.071169969

A class should not be forced to implement an interface that it doesn't use.

```ruby
class Movie
  def play
    puts "Playing movie"
  end

  def increase_volume
    puts "Increasing volume"
  end
end

class ModernTimes < Movie
  def increase_volume
    raise
  end
end
```

```ruby
class Movie
  def play
    puts "Playing movie"
  end
end

class VolumeControl
  def increase_volume
    puts "Increasing volume"
  end
end

class TheLionKing
  include Movie
  include VolumeControl
end

class ModernTimes
  include Movie
end
```
