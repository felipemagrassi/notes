# Logging at crashes - 2023-05-15 00:35:45.974037963 -0300 -03 m=+3122.805809937

We can crash a #ruby program with `raise` or `fail` and #log
the error with `at_exit`.

**[ruby\_variables](https://ruby-doc.org/docs/ruby-doc-bundle/Manual/man-1.4/variable.html#dollar)**

```ruby
require "yaml"

module CrashLogger
  def self.log_crash_info(error=$!)
    program_name = $0
    process_id = $$
    timestamp = Time.now.utc.strftime("%Y%m%d-%H%M%S")

    filename = "/home/felipemagrassi/robust_ruby/crash-#{process_id}-#{timestamp}.yml"
 
    error_info = {}
    error_info["error"] = error
    error_info["stacktrace"] = error.backtrace
    error_info["environment"] = ENV.to_h

    File.write(filename, error_info.to_yaml)
    filename
  end
end

at_exit do 
  unless $!.nil? || $!.is_a?(SystemExit)
    filename = CrashLogger.log_crash_info
    $stderr.puts "A crash log has been saved to: #{filename}"
  end
end

1/0
```
