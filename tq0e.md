# Redo in #ruby - 2023-05-15 00:33:18.689319745 -0300 -03 m=+2975.521091709

Redo restart the current block back up to the beginning, but unlike next, does not advance to the next iteration

```ruby
FILES = %W[file1 file2 file3 file4 file5 file6 file7 file8 file9]

MIRRORS = %W[a.example.org b.example.org c.example.org]

require "net/http"
require "webmock"

include WebMock::API
WebMock.enable!

request_count = 0
err = {status: 502}
ok  = {status: 200, body: "OK!"}
stub_request(:get, /.*.example.org.*/)
  .to_return(->(r){ request_count += 1; request_count % 4 == 0 ? err : ok })


FILES.each do |file|
  mirror = MIRRORS.first
  uri = URI("http://#{mirror}/#{file}")
  puts "Requesting #{uri}"
  result = Net::HTTP.get_response(uri)
  if result.code == "200"
    puts "Success!"
  else
    puts "Error #{result.code}; switching mirrors"
    MIRRORS.shift
    redo
  end
end
```

*Results:*

    Requesting http://a.example.org/file1
    Success!
    Requesting http://a.example.org/file2
    Success!
    Requesting http://a.example.org/file3
    Success!
    Requesting http://a.example.org/file4
    Error 502; switching mirrors
    Requesting http://b.example.org/file4
    Success!
    Requesting http://b.example.org/file5
    Success!
    Requesting http://b.example.org/file6
    Success!
    Requesting http://b.example.org/file7
    Error 502; switching mirrors
    Requesting http://c.example.org/file7
    Success!
    Requesting http://c.example.org/file8
    Success!
    Requesting http://c.example.org/file9
    Success!
