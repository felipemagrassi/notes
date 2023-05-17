# Log rotation in ruby - 2023-05-14 23:51:50.377145156 -0300 -03 m=+487.208917120

## #log rotation in #ruby

Aging: Time-based logging deletion

```ruby
require "logger"
logger = Logger.new("logger.log", "daily")

require "timecop"

logger.info "Hello today"

Timecop.travel(Date.today + 1)

logger.info "Hello Tomorrow"

Timecop.travel(Date.today + 2) 

logger.info "Hello the next day"
```

Rotation: Size-limited logging deletion

```ruby
require "logger"
number_of_log_files_to_keep = 4
size_cutoff_in_bytes = 2048
logger = Logger.new("logger.log", number_of_log_files_to_keep, size_cutoff_in_bytes)
```


