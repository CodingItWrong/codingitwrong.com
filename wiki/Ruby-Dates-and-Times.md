---
title: Ruby Dates and Times
---

Tips on dealing with dates and times in [Ruby](Ruby). Note: [Rails](Rails)' Active Support gem provides a number of [additional date and time facilities][ActiveSupport].

- [Date][Date]: date only
- [Time][Time]: dates and times, but doesn't handle historic calendar reforms
- [DateTime][DateTime]: dates and times, and handles historic calendar reforms

Newer versions of Ruby, Time and DateTime are mostly the same.

ActiveRecord uses DateTime.

## Specific Needs

- Get a specific date/time in the current server time zone: `DateTime.now.beginning_of_day.change(year: 2016, month:8, day:26)`

[ActiveSupport]: http://api.rubyonrails.org/classes/DateTime.html
[Date]: http://ruby-doc.org/stdlib-2.3.1/libdoc/date/rdoc/Date.html
[DateTime]: http://ruby-doc.org/stdlib-2.3.1/libdoc/date/rdoc/DateTime.html
[Time]: http://ruby-doc.org/stdlib-2.3.1/libdoc/time/rdoc/Time.html
