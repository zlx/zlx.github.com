---
layout: post
title: "captcha with Devise"
date: 2015-08-27 15:49
comments: true
category: tech
tags: devise simple_captcha2 bots fail_attempts
---

现在网络上各种扫描器和网络爬虫越来越泛滥，如何让你的网站变得更强壮，以抵御这些不速之客呢？

验证码作为一个简单而又有效的解决方案，很好的将机器人和人类区分开来。呃~~, 当然，未来可能就不一定了。

目前主流的验证码形式有这么几种：

<!--more-->

1. 问答题
2. 照片验证码
3. 图片验证码

第一种比较简单直接，它主要的问题是需要存储大量的数据，理论上题库越大越难以破解。这里有一个实现: [humanizer](https://github.com/kiskolabs/humanizer)，有兴趣的同学可以看看

第二种是利用现实中的照片，人类识别照片很容易，机器人可不容易，日PV巨大的某数字网站就是采用这种做法屏蔽了大量的爬虫，勉强维持着僧多粥少的局面。这类方案最出名的还是 Google 推出的 [recaptcha 服务](https://www.google.com/recaptcha)，[recaptcha](https://github.com/ambethia/recaptcha) 就是是基于它的一个实现。

这里我们重点介绍一下第三种实现，它的原理是根据提供的字符组合，生成一张图片，然后验证用户的输入来判断是不是人类。从图形学的角度来说，图片可以做得非常复杂，使得机器识别的难度非常大。典型的实现是 [simple_captcha2](https://github.com/pludoni/simple-captcha)


---


环境说明：以下实例基于 ruby 2.1 + rails 3.2.22 + simple_captcha2 0.2.2 + devise 3.1.0


#### Devise 准备

首先，根据 Devise 的文档，配置好你的项目，确保能够正常注册和登录。

然后从 Devise::SessionsController 继承一个你自己的 SessionController, 如下：

    class Users::SessionsController < Devise::SessionsController
      ...
      def create
        super
      end
      ...
    end

可以参考：[configuring-controllers](https://github.com/plataformatec/devise/tree/v3.1.0#configuring-controllers)。

接着配置路由如下:

    devise_for :users, :controllers => { :sessions => "users/sessions" }

然后运行命令：

    rails generate devise:views users
    
删除 sessions 以外的目录。可以参考 [configuring-views](https://github.com/plataformatec/devise/tree/v3.1.0#configuring-views)

现在你再尝试登录一下，应该没有破坏代码。

#### SimpleCaptcha2 准备

好了，接下去我们按照 [simple_captcha2](https://github.com/pludoni/simple-captcha/tree/v0.2.2) 的文档安装好依赖的库。

并执行：

    rails generate simple_captcha
    rails db:migrate

创建好需要的数据库和配置文件。你可以参考官方文档配置验证码的难度及样式等.


#### Controller Based 验证码


接下来我们面临一个抉择：

浏览 simple_captcha2 的文档，你会发现它提供了两种方式来验证，一种是 Controller Based，一种是 Model Based。 前一种依赖于 session 来存储数据，后一种依赖 Model 来存储数据。

你可以根据你的实际情况来抉择，我这里选择了 Controller Based 做介绍。

第一步，我们先把验证码显示出来：

在 `sessions/new.html.erb` 合适位置加上:

    ...
    <%= f.input :captcha do %>
      <%= show_simple_captcha %>
    <% end -%>
    ...

我这里是 erb 配合了 simple_form, 你可以根据实际情况调整语法格式。

修改 `Users::SessionsController`, 加上验证：

    class User::SessionsController < Devise::SessionsController
      include SimpleCaptcha::ControllerHelpers
      ...
      def create
        if simple_captcha_valid?
	      super
	    else
	      flash[:alert] = "Captcha code is wrong, try again."
	      self.resource = resource_class.new(sign_in_params)
	      respond_with_navigational(resource) { render :new }
	    end 
      end
      ...
    end

OK, 刷新页面，你应该可以看到验证码了。

***但是***

提交一下试试，啊，不对劲！我相信你也发现了，验证码好像无效，未填写验证码也能通过验证。

这是 Devise 的一个*神奇之处*，其实它在进入 `Sessions#create` 已经验证通过了，所以并没有执行验证码的验证代码，具体讨论可以查看这里 [Github - plataformatec/devise#2106](https://github.com/plataformatec/devise/issues/2106)

我们在 Users::SessionsController 里面加上这句：

    skip_before_filter :require_no_authentication, :only => [:create]

现在再试试，应该能够正常使用验证码检查是不是机器人了。


### 最大失败次数

虽然我们要抵御机器人，但是我们还是希望正常用户直接登录即可，不需要每次都瞪大眼睛输入验证码，这可能会让他们感觉很不爽。

那么，我们做一个限制：尝试登录失败三次，再显示验证码，成功登录则重新计数。

#### 数据存储

那么，问题来了，这个数据存在哪里呢？

这个数据跟用户直接相关，存在 cookie 里面？这样服务器没有压力了。但是机器人不会给你一个保持 Cookie 的，所以这招对机器人就失效了。

好吧，那只能存在服务端，直接存在用户表里面，正好和和用户一一对应；亦或者存在缓存中，使用一个和用户直接相关的 Key。

貌似两种方案都可以，我们且不做抉择，继续往下考虑：

1. 假如用户失败了几次，然后放弃了，过了很长时间后再次登录，这时候我们需要验证码吗？
2. 如果某一个机器人一直在尝试用一个不存在的用户名字暴力破解，我们需要验证码吗？

如果你两个回答都是 YES， 那你最好选择缓存来保存这个数据。

#### 计数器

我们希望：

1. 用户登录失败时把当前用户的计数加 1
2. 用户登录成功后把计数清零
3. 在超过一定时间计数自动清零

这里我们将这几个相关的处理放到同一个 class 里面处理, 定义如下 Class:


    class UserLoginFailCounter
	  include Singleton
	  EXPIRED_TIME = 10.minutes
	  FAILED_ATTEMPTS = 3
	
	  def increment(login)
	    count = current_count(login) + 1
	    cache.write(key_for(login), count, expires_in: EXPIRED_TIME)
	    count
	  end
	
	  def reset(login)
	    cache.delete(key_for(login))
	  end
	  
	  def show_captcha?(login)
	    current_count(login) >= FAILED_ATTEMPTS
	  end
	
	  def current_count(login)
	    cache.read(key_for(login)) || 0
	  end
	
	  def key_for(login)
	    "UserLoginFailCount:#{login}"
	  end
	
	  private
	
	  def cache
	    Rails.cache
	  end
	
	end

另外 Rails 默认的缓存实现不支持过期的功能，这边需要配合 MemoryStore 或者 RedisStore 等缓存实现，具体请参考其它文章。

然后修改 `sessions/new.html.erb`, 加上 `show_captcha?` 判断

    ...
    <% if UserLoginFailCounter.instance.show_captcha?(resource.login) -%>
      <%= f.input :captcha do %>
        <%= show_simple_captcha %>
      <% end -%>
    <% end -%>
    ...

修改 Users::SessionsController：

    class User::SessionsController < Devise::SessionsController
      ...
      def create
        if verify_captcha?
          super
	      UserLoginFailCounter.instance.reset(sign_in_params["login"])
	    else
          flash[:alert] = "Captcha code is wrong, try again."
	      self.resource = resource_class.new(sign_in_params)
	      respond_with_navigational(resource) { render :new }
	      UserLoginFailCounter.instance.increment(sign_in_params["login"])
	    end 
      end
      ...
      private
      
      def verify_captcha?
        !UserLoginFailCounter.instance.show_captcha?(sign_in_params["login"]) || simple_captcha_valid?
      end
    end

看上去 OK 了，如果你也这样觉得，那你就是图样图森破了。

当你打开页面，重复几次失败的登录，满怀期待地期待验证码出现的时候， Duang!!!

请求执行到 `super` 这一行后离奇地失踪了，后面的代码不执行，也没有异常抛出来，Devise 默默的把事情做完了。好开心，有么有！！

话说回来，要解决这个问题，在 `devise.rb` 配置文件里面加上:

     Warden::Manager.before_failure do |env, opts|
	   user_params = env["action_dispatch.request.request_parameters"]["user"]
	   UserLoginFailCounter.instance.increment(user_params["login"]) if user_params.is_a?(Hash) && user_params["login"].present?
	 end

现在再去试试吧。

---

### 完整代码

#### sessions#new.html.erb

    <% if UserLoginFailCounter.instance.show_captcha?(resource.login) -%>
      <%= f.input :captcha do %>
        <%= show_simple_captcha %>
      <% end -%>
    <% end -%>
    
#### users/sessions_controller.rb

    class Users::SessionsController < Devise::SessionsController
	  include SimpleCaptcha::ControllerHelpers
	  skip_before_filter :require_no_authentication, :only => [:create]
	
	  def create
	    if verify_captcha?
	      super
	      reset_fail!
	    else
	      set_captcha_error
	      increment_fail!
	      response_to_new
	    end
	  end
	
	  private
	
	  def response_to_new
	    self.resource = resource_class.new(sign_in_params)
	    respond_with_navigational(resource) { render :new }
	  end
	
	  def verify_captcha?
	    !UserLoginFailCounter.instance.show_captcha?(sign_in_params[:login]) || simple_captcha_valid?)
	  end
	
	  def increment_fail!
	    UserLoginFailCounter.instance.increment(sign_in_params[:login])
	  end
	
	  def reset_fail!
	    UserLoginFailCounter.instance.reset(sign_in_params[:login])
	  end
	
	  def set_captcha_error
	    flash[:alert] = "Captcha code is wrong, try again."
	  end
	
	end
	
#### user_login_fail_counter.rb

    class UserLoginFailCounter
	  include Singleton
	  EXPIRED_TIME = 10.minutes
	  FAILED_ATTEMPTS = 3
	
	  def increment(login)
	    count = current_count(login) + 1
	    cache.write(key_for(login), count, expires_in: EXPIRED_TIME)
	    count
	  end
	
	  def reset(login)
	    cache.delete(key_for(login))
	  end
	  
	  def show_captcha?(login)
	    current_count(login) >= FAILED_ATTEMPTS
	  end
	
	  def current_count(login)
	    cache.read(key_for(login)) || 0
	  end
	
	  def key_for(login)
	    "UserLoginFailCount:#{login}"
	  end
	
	  private
	
	  def cache
	    Rails.cache
	  end
	end    

#### devise.rb

    Devise.setup do |config|
      Warden::Manager.before_failure do |env, opts|
	    user_params = env["action_dispatch.request.request_parameters"]["user"]
	    UserLoginFailCounter.instance.increment_fail(user_params["login"]) if user_params.is_a?(Hash) && user_params["login"].present?
	  end
	end

enjoy!
