---
layout: post
title: "有趣的 after_touch"
date: 2013-05-10 21:01
comments: true
category: tech
tags: rails after_touch callbacks
---

`Rails` 里面存在很多 `callbacks`， 比如 `before_save`, `after_save`,
`before_create`, `after_create`, `before_destroy`, `after_destroy`
等等。这些 `callbacks` 让我们的程序变得更加方便和灵活。 [这里](http://guides.rubyonrails.org/active_record_validations_callbacks.html#available-callbacks)
提供了官方的使用建议和指南。

我之前遇到这样一个需求：我有一个专题，专题里面有许多书籍。专题需要发布到不同平台上面，然后，每次更新专题时，我要更新各个平台上的专题，保证专题里面的书籍都发布到了该平台上面。

<!--more-->

我就使用了一个 `callback` 来发布相关书籍：


    class Topic < ActiveRecord::Base
      has_many :apps_topics
    end

    class AppsTopic < ActiveRecord::Base
      belongs_to :topic
      after_create :pub_books

      def pub_articles
    	topic.articles.each do |article|
    	  AppArticle.where(article_id: article.id, app_id: app_id).first_or_create
    	end
      end
    end

但是当更新专题（`topic`)时，`pub_articles` 这个 `callback` 并不会执行。

### 如何解决这个问题？

在保存专题的地方加上：

    @topic.apps_topics.map(&:touch)

然后加上一句：

    class AppsTopic < ActiveRecord::Base
      belongs_to :topic
      after_create :pub_books
      ***after_touch :pub_books***

      def pub_articles
    	topic.articles.each do |article|
    	  AppArticle.where(article_id: article.id, app_id: app_id).first_or_create
    	end
      end
    end

这样，每次更新 topic 时候，都会更新 apps_topics, 然后 apps_topic 会检查一遍专题里面的书籍是否都已经发布，如果发布，则发布到对应平台上。


### 但是，after_touch 这个方法哪里来的呢？官方文档里面没有介绍

看看 [ActiveRecord CallBacks](https://github.com/rails/rails/blob/master/activerecord/lib/active_record/callbacks.rb) 里面时怎么说的:

{% gist 1187700 active-record-callbacks.rb %}


## 参考资料：

[了解 ActiveSupport 里面的回调](http://thomasmango.com/2011/09/02/getting-to-know-active-support-callbacks/)


enjoy!
