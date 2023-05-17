# Aliases in IRB - 2023-05-15 00:28:15.908804667 -0300 -03 m=+2672.740576641

add the following lines to the .irbrc in the root of your #rails app:

```ruby
def me 
  @me ||= User.find_by_email('user@test.com')
end

User = Organization::User
Group = Organization::Group
```

add the following lines to the .irbrc in your home folder:

```ruby
require 'irb/ext/save-history'

IRB.conf[:SAVE_HISTORY] = 10000
IRB.conf[:HISTORY_FILE] = "#{ENV['HOME']}/.irb-history"
```

then use lazyme to create a #alias to your rails app:

```bash
gem install lazyme
lazyme ~/.irb-history
```
