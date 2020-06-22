* ### Redo rails migration
```bash
bundle exec rake db:migrate:redo VERSION=20200312143950
```

* ### Flush sidekiq DB
```ruby
Sidekiq.redis { |conn| conn.flushdb }
```
