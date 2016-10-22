---
published: true
title: Rails 里面的时间
layout: post
comments: true
categories: tech
tags: rails, time
---

***特别注意：以下方法均是在 Ruby 2.2.0p0 + Rails 4.2.6 环境下测试，不保证在其他环境下正确。***

## Rails 里面配置时区的时间

- Date.current
- Time.current
- DateTime.current
- Time.zone.now
- Time.zone.today
- Time.now.in_time_zone
- 2.days.since
- 3.weeks.ago
- Time.zone.parse("2015-07-04 17:05:37")
- DateTime.strptime('2001-02-03T04:05:06+07:00', '%Y-%m-%dT%H:%M:%S%z').in_time_zone
- Time.strptime('2001-02-03T04:05:06+07:00', '%Y-%m-%dT%H:%M:%S%z').in_time_zone

## 系统时区的时间

- Date.today
- Time.now
- DateTime.now
- Date.today.to_time
- Time.parse("2015-07-04 17:05:37")

## 其它时区时间

- Time.now.utc
- DateTime.now.utc
- Time.strptime('2001-02-03T04:05:06+07:00', '%Y-%m-%dT%H:%M:%S%z')
- DateTime.strptime('2001-02-03T04:05:06+07:00', '%Y-%m-%dT%H:%M:%S%z')

有什么新的发现，随时更新

参考文章

- [It's About Time (Zones)](https://robots.thoughtbot.com/its-about-time-zones)
- [Working with time zones in Ruby on Rails](http://www.elabs.se/blog/36-working-with-time-zones-in-ruby-on-rails)
