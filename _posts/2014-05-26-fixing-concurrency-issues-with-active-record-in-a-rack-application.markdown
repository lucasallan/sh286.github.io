---
layout: post
title:  "Fixing ActiveRecord concurrency issues in a rack application (without Rails)"
date:   2014-05-26 16:49:00
---

In the last 6 months we have been working to split out main application into small isolated service applications.

As part of this strategy we need an application that it's going to provide information about some services -
all the applications will access it through an internal API.
This is going to be a small application to respond to http request and read/write data into a database.

While Rails it's a great solution for mostly of the applications, doesn't make any sense to use Rails for a small application like that.

Although Ruby it's still my favorite language and ActiveRecord is a great ORM solution -
so doesn't make any sense to not use Ruby either.


### [Rack][rack], [Grape][[grape] and [ActiveRecord][ar]

We have decided to build a Rack application with ActiveRecord, although [Rack][rack] doesn't provide a good DSL to build your HTTP endpoints. For our luck, some fellows (from [Intridea][intridea]) built a great API
framework on top of Rack. [Grape][grape] is a great and powerful micro-framework for creating REST-like APIs in Ruby.

Everything were working just fine and running with [JRuby][jruby] and [Puma][puma]. Until as part of our QA process we did several requests to an API endpoint and then:

```
[2014-05-26T13:54:00.898000 #99234]  WARN -- : could not obtain a database connection within 5.000 seconds (waited 5.001 seconds)
active_record/connection_adapters/abstract/connection_pool.rb:190:in `wait_poll'
org/jruby/RubyKernel.java:1501:in `loop'
/active_record/connection_adapters/abstract/connection_pool.rb:181:in `wait_poll'
active_record/connection_adapters/abstract/connection_pool.rb:136:in `poll'
/Users/lucasallan/.rbenv/versions/jruby-1.7.12/lib/ruby/1.9/monitor.rb:211:in `mon_synchronize'
...
org/jruby/RubyProc.java:271:in `call'
/Users/lucasallan/.rbenv/versions/jruby-1.7.12/lib/ruby/gems/shared/gems/puma-2.8.2-java/lib/puma/thread_pool.rb:92:in `spawn_thread'
127.0.0.1 - - [26/May/2014 13:54:00] "POST /domains HTTP/1.1" 500 82 5.0530
```
After spent some time digging into it I figured out that the exception happens because for our application it's not releasing the db connection after use it.

I tried some solutions like use `ActiveRecord::Base.clear_active_connections!`
inside a `after` block after each request. But it didn't work.

Although, active_record does have a way to close the connection and return it to the connection pool: `ActiveRecord::Base.connection_pool.with_connection`

So if I move all my code to a block and pass it to `ActiveRecord::Base.connection_pool.with_connection` then I won't have any issue.
Example:

```
ActiveRecord::Base.connection_pool.with_connection do
  User.update_attribute(:name, 'lucas')
end
```

Unfortunately it's not very handy to pass all my ActiveRecord calls to this block.

I can easily forget to pass it and every time this code being executed I'd lose one of my connections and after a few times
I will end up running out of db connections and an exception will be raised.

So to fix it, I created a simple monkey patch that will make all the active_records calls use this block by default:

```Ruby
module ActiveRecord
  class Base
    singleton_class.send(:alias_method, :original_connection, :connection)

    def self.connection
      ActiveRecord::Base.connection_pool.with_connection do |conn|
        conn
      end
    end
  end
end
```

For more information, visit: [http://api.rubyonrails.org/classes/ActiveRecord/ConnectionAdapters/ConnectionPool.html][ar-doc]

[grape]: https://github.com/intridea/grape
[rack]: http://rack.github.io/
[intridea]: http://www.intridea.com/
[ar]: http://api.rubyonrails.org/classes/ActiveRecord/Base.html
[jruby]: http://jruby.org
[puma]: http://puma.io/
[ar-doc]: http://api.rubyonrails.org/classes/ActiveRecord/ConnectionAdapters/ConnectionPool.html
