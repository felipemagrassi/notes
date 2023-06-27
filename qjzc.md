# Liskov Substitution Principle - 2023-06-25 15:17:18.620971615 -0300 -03 m=+1491.528216480

tags: #OOP #SOLID #DesignPatterns

Subclasses should be substitutable for their base classes.

```ruby

class Movie
  def increase_volume
  end
end

class LordOfTheRings < Movie
  def increase_volume
  end
end

class ModernTimes < Movie
  def increase_volume
  end
end
```

As modern times is a silent movie, it doesn't make sense to increase its volume. So, we can't substitute a Movie for a ModernTimes.
