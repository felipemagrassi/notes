# Single Responsibility Principle - 2023-06-25 14:52:27.112298598 -0300 -03 m=+0.019543453

tags: #SOLID

One class should one have one responsibility. If a class have more than one reason to change,
it should be split into two or more classes.

Example: Rich entity model that connects to the database and also has business logic.

```ruby
class Course
  attr_acessor :name, :description, :category

  def initialize()
    @name = ''
    @description = ''
    @category = ''
  end

  def create_course()
    Course.create()
  end

  def create_category()
    category = Category.create()

    self.category = category
  end
end
```

```ruby
class Course
  def initialize(name, description)
    @name = name
    @description = description
  end

  attr_reader :name, :description
end

class CourseCreator
  def create_course(name, description)
    CourseRepository.create(name, description)
  end
end

class CourseRepository
  def self.create(name, description)
    Course.create(name: name, description: description)
  end
end
```
