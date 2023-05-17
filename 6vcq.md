# Removing dead code - 2023-05-15 00:23:58.959214813 -0300 -03 m=+2415.790986777

Dead-code is code that used to be used, but is no longer used. It can be a method no longer called, an unreachable return statement. #Dead-code has no upside and costs money, time and maintenance.

In smaller projects, is easier to identify dead code, but in larger projects, it can be a challenge. A few tools can aid us in analyzing, and they could be splitted in two categories:

*   **Static analysis:** tools that analyze the code without running it. They are fast, but they can't detect all dead code. They are also limited by the language they support.

    *   [Debride](https://github.com/seattlerb/debride)
    *   [Unused](https://github.com/unused-code/unused)

*   **Dynamic analysis:** tools that run the code to detect dead code. They are slower, but they can detect more dead code. They are also limited by the language they support.

    *   [Coverband](https://github.com/danmayer/coverband)
    *   [Scythe](https://github.com/michaelfeathers/scythe)

None of these tools are perfect, and they all have their own limitations. But they can be used together to get a better picture of the dead code in our codebase.

A deeper investigation must be done to ensure that the code is indeed dead. It could be that the code is used in a different environment, or that it is used in a different way than we expect.

We can use [git pickage](http://www.philandstuff.com/2014/02/09/git-pickaxe.html) to find the commit that introduced the dead code, then create a branch with proposed-as-dead commits for easy traceback

## Verifying dead code with a logger module

We can use a logger module to verify if the dead code is indeed dead. We can use a custom logger tag to filter the logs, and we can use a custom token to ignore the dead code in the coverage report.

```ruby
  module DeadCodeLogger
    class << self
      def log(pr_link, &exec_block)
        exec_block.call
      ensure
        Rails.logger.tagged('proposed-dead-code') do |logger|
          logger.info "Proposed as dead code in #{pr_link} and executed at #{exec_block.source_location}"
        end
      end
    end
  end

    # :dead-code:
  def foo
    DeadCodeLogger.log 'https://github.com/username/sample-app/pull/1' do
      return 'bar';
    end
  end
  # :dead-code:

SimpleCov.start do
  nocov_token 'dead-code'
end
```
