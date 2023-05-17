# Circuit break in ruby - 2023-05-15 00:37:29.879065637 -0300 -03 m=+3226.710837591

tags: #ruby #circuit-breaker

```ruby
class CircuitBreaker
  class OpenError < StandardError
  end

  def initialize(threshold: 5)
    @state       = :closed
    @threshold   = threshold
    @error_count = 0
  end

  def monitor
    raise OpenError, "Circuit breaker is open" if @state == :open
    result = yield
    handle_success
    result
  rescue OpenError
    raise
  rescue => error
    handle_error(error)
    raise
  end

  def open?
    @state == :open
  end

  private

  def handle_success
    @error_count = 0
  end

  def handle_error(error)
    @error_count += 1
    warn "Failure count is now #{@error_count}"
    if @error_count >= @threshold
      @state = :open
      warn "Circuit breaker has tripped!"
    end
  end
end
```
