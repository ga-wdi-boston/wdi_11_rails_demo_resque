# Resque Demo

Resque is used for background tasks, such as checking an API, sending e-mail, doing a heavy database calculation, etc.

Resque stores its tasks in Redis, a key-value database that can be thought of as a giant hash table. To make Resque work, we need to ensure that Redis is installed

OS X:

    brew install redis
    redis-server

Ubuntu

    apt-get install redis
    redis-server

In another tab start

    redis-cli monitor

To create a job, simply create a module that will be loaded by Rails.

```ruby
    class JobName
      @queue = :default

      def self.perform
        sleep 1
        puts "This job just waits one second. Code here is executed outside of Rails"
        puts "If the worker has access to the environment variables,"
        puts "it can access anything a Rake task could."
      end
    end
```

Configure Resque to use the proper Redis connection by putting the following in `config/initializers/resque.rb`
    
    if ENV["REDISCLOUD_URL"]
      uri = URI.parse(ENV["REDISCLOUD_URL"])
      RedisClient = Redis.new(:host => uri.host, :port => uri.port, :password => uri.password)
    else
      RedisClient = Redis.new(host: 'localhost')
    end
    
    Resque.redis = RedisClient


Then to queue up your code, run `Resque.enqueue(JobName)`

To start the workers, you'll have to run another process outside of `rails server`. To do this, run `resque work` in your terminal.

Now, you can't just hop over to a Heroku terminal and run `resque work` so we need to use a *Procfile* to make that work. Create a file called `Procfile` with the following content. This line can be run with the `foreman` app, which basically runs an application as Heroku would run it.

    web: bundle exec rails s
    worker: env QUEUE=* bundle exec resque work

One thing to note is that the [primary Resque documentation](https://github.com/resque/resque) doesn't line up with the code. I have made a [fork of the code for better documentation](https://github.com/tibbon/resque/tree/documentation_cleanup), but I haven't updated it in a while.
