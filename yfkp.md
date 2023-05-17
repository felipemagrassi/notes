# Better performance with log - 2023-05-14 23:49:58.511354283 -0300 -03 m=+375.343126257

Using #logging can decrease the application #performance or increase costs even when `log.level` is not set for debug/info usage. The LOC is still built  even when the log doesn't hit desidered output

Possible Solution, passing a block to Logger library:

```ruby
  logger.debug { "Request from user: current_user[:name]"} # better
```
vs

```ruby
  logger.debug "Request from user: current_user[:name]"
```

The string argument is evaluated and interpolated before goes to the logger, while the block is after, so if the log_level is beyond debug level, it won't reach the LOC


