---
layout: post
title: "Sidekiq 和 Connection Pool"
date: 2014-04-04 21:49
comments: true
category: tech
tags: Rails Sidekiq ConnectionPool
---

## 缘起

由于项目需要，需要在使用 Sidekiq 来处理后台任务，为了保证 Sidekiq 能够及时处理任务，我们给它配置了比较高的并发数：

    :concurrency: 5
    :pidfile: tmp/pids/sidekiq.pid
    staging:
      :concurrency: 100
    production:
      :concurrency: 100

相对应的，我们需要配置数据库连接池来防止数据库连接不够用，导致连接超时：`ConnectionTimeoutError: no connection can be obtained from the pool.`

一般情况下，你可以直接在 `config/database.yml` 配置:

    pool: 108

但这样会导致一个问题：如果你和我们一样使用 [unicorn](http://unicorn.bogomips.org/) 这种多进程 web server, 并开了多个进程的话，这就会导致你整个 Rails Application 会使用 108 * [unicorn#size] 个数据库连接，这就造成了很大浪费。

###那么，如何给 Sidekiq 单独配置 pool 而避免影响 unicorn 的数据库连接呢？###

<!--more-->

如果你想直接获取答案，请直接查看[解决模块]()。

## 剖析

首先我们先来了解一下 Rails 里面数据库连接池的工作原理。

Rails 在启动工程中会加载 ActiveRecord, 所以会执行下面这段代码：

    initializer "active_record.initialize_database" do |app|
     ActiveSupport.on_load(:active_record) do
       self.configurations = app.config.database_configuration || {}
       establish_connection
     end
    end

这里首先设置了 configurations , 然后调用 establish_connection 来初始化数据库连接。如果你深入 database_configuration 可以发现它会去读取 config/database.yml 文件。

    def establish_connection(spec = ENV["DATABASE_URL"])
      resolver = ConnectionAdapters::ConnectionSpecification::Resolver.new spec, configurations
      spec = resolver.spec

      ...

      remove_connection
      connection_handler.establish_connection self, spec
    end  

进入 establish_connection, 你可以看到它默认接受 ENV["DATABASE_URL"] 作为参数，

然后调用 ConnectionAdapters::ConnectionSpecification::Resolver.new(spec, configurations) 来创建一个 resolver 对象，然后调用 Resolver#spec

    class Resolver
        def initialize(config, configurations)
         @config         = config
         @configurations = configurations
        end
        
        def spec
         case config
         when nil
           raise AdapterNotSpecified unless defined?(Rails.env)
           resolve_string_connection Rails.env
         when Symbol, String
           resolve_string_connection config.to_s
         when Hash
           resolve_hash_connection config
         end
        end
    end   

在 Resolver 里面，可以看到它根据 config 进行了三种不同处理，分别对应了 establish_connection 不传参数，接受 ENV["DATABASE_URL"] 参数和接受 hash 类型的参数三种情况。

而在 resolve_string_connection 里面，

    def resolve_string_connection(spec) # :nodoc:
     hash = configurations.fetch(spec) do |k|
       connection_url_to_hash(k)
     end
     ...
     resolve_hash_connection hash
    end

我们可以看到，它首先在 configurations 里面搜索对应环境的配置信息，如果没有，就调用 connection_url_to_hash 来生成，最终有调用 resolve_hash_connection 来统一处理配置。

    def connection_url_to_hash(url) # :nodoc:
     config = URI.parse url
     adapter = config.scheme
     adapter = "postgresql" if adapter == "postgres"
     spec = { :adapter  => adapter,
              :username => config.user,
              :password => config.password,
              :port     => config.port,
              :database => config.path.sub(%r{^/},""),
              :host     => config.host }

     spec.reject!{ |_,value| value.blank? }

     uri_parser = URI::Parser.new

     spec.map { |key,value| spec[key] = uri_parser.unescape(value) if value.is_a?(String) }

     if config.query
       options = Hash[config.query.split("&").map{ |pair| pair.split("=") }].symbolize_keys

       spec.merge!(options)
     end

     spec
    end
    
打开 connection_url_to_hash 方法，我们可以看出，它处理了 ENV["DATABASE_URL"] 作为参数的情况。

*至此，我们明白，可以通过配置合适的参数，就可以让 Sidekiq 自己使用单独的 pool 而不影响 unicorn 进程。*

## 解决

对于我们的问题，我们可以在 sidekiq 的初始化文件中修改这个配置，比如：

    Sidekiq.configure_server do |config|
      ...
      sconfig = Rails.application.config.database_configuration[Rails.env]
      sconfig['pool']            = 100
      ActiveRecord::Base.establish_connection sconfig
      
      # 或者 
      
      ActiveRecord::Base.configurations[Rails.env]['pool'] = 100
      ActiveRecord::Base.establish_connection

      ... 
    end

抑或是直接在启动 Sidekiq 进程的时候传递 ENV["DATABASE_URL"] 参数:

    DATABASE_URL="mysql2://root:@localhost/yourdatabasename?pool=100" bundle exec sidekiq -C config/sidekiq.yml       

另外更多详细的数据库配置参数，请参考：[ConnectionPool#Options](http://api.rubyonrails.org/classes/ActiveRecord/ConnectionAdapters/ConnectionPool.html#class-ActiveRecord::ConnectionAdapters::ConnectionPool-label-Options)

### 参考资料

1. [Sidekiq Config Pool Only for Sidekiq](https://github.com/mperham/sidekiq/wiki/Advanced-Options#rails-32-and-newer)
2. [Sidekiq Issue#custom ActiveRecord Connection Pool Size](https://github.com/mperham/sidekiq/issues/503)
3. [concurrency and database connections](https://devcenter.heroku.com/articles/concurrency-and-database-connections)

enjoy!
