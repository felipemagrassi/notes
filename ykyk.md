# Using ruby at exit - 2023-05-15 00:00:01.869278679 -0300 -03 m=+978.701050693

`at_exit` allow us to run a block of code when the program exits in #ruby

*   `at_exit` runs as a stack -> LIFO
*   does not work with `exit!`
*   good for closing files and connections before finishing the program

```ruby
at_exit { puts "Bye" }
at_exit { puts "Bye Bye" }
at_exit { puts "Bye Bye Bye" }

puts "Hello!"
```

*Results:*
```
Hello!
Bye Bye Bye
Bye Bye
Bye
No examples found.


Finished in 0.00019 seconds (files took 0.02815 seconds to load)
0 examples, 0 failures

```




