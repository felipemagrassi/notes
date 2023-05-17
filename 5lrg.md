# using ruby default logger - 2023-05-14 23:43:43.181510822 -0300 -03 m=+0.013282786

#logging with #ruby default #logs gem

## Basic logging

```ruby
require 'logger'

log = Logger.new($stderr) # log to console
# log = Logger.new("test.log") # log to file

log.level = Logger::DEBUG # sets level to debug

logger.datetime_format = "%c"
logger.formatter = ->(severity, datetime, progname, message) { 
  {
    severity: severity,
    time: datetime,
    message: message
  }.to_json + "\n"
}

log.debug "This is a debug message"
log.info "This is a info message"
log.warn "This is a warn message"
log.error "This is an error message"
log.fatal "This is a fatal message"
log.unknown "Unknown level"
```
