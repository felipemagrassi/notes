# Adding color to log 🔵🔴🟠 - 2023-05-14 23:55:33.668176615 -0300 -03 m=+710.499948589

adding color to #log stdout

```ruby
require "pastel"
require "logger"

pastel = Pastel.new

colors = {
  "FATAL" => pastel.red.bold.detach,
  "ERROR" => pastel.red.detach,
  "WARN" => pastel.yellow.detach,
  "INFO" => pastel.green.detach,
  "DEBUG" => pastel.white.detach
}

logger = Logger.new($stdout)

logger.formatter = ->(severity, datetime, progname, message) {
  colorizer = $stdout.tty? ? colors[severity] : ->(s){s}
  "#{colorizer.(severity)}: #{message}\n"
}

logger.fatal "teste" 
```


