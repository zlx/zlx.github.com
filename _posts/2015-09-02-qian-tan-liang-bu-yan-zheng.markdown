---
layout: post
title: "浅谈两步验证"
date: 2015-09-02 10:41
comments: true
category: tech
tags: 
---

两步验证，即 [2FA](https://en.wikipedia.org/wiki/Two-factor_authentication), 是近年来被广泛使用的技术，许多国外的大厂和有节操的公司都已经用上了，且看[这个列表](https://en.wikipedia.org/wiki/Google_Authenticator#Usage)。

今天我们就来看看这个时髦玩意到底是什么东西，这篇文章主要包括以下四部分：

1. 两步验证的使用场景
2. 两步验证的原理
3. 两步验证的安全性
4. 两步验证的实践

<!--more-->

### 两步验证的使用场景

广泛的两步验证是 1984 年就提出来的一个概念，目前流行的主要是基于智能手机的两步验证，其中主流的又分 APP 方式和短信方式。

基本流程是：用户使用账号密码登录网站后，网站通过某种方式通知用户一个 CODE，用户把这个 CODE 输入网站验证，验证通过才允许登录。

这样在原先用户名和密码基础上加上一步验证，进一步保证了安全性。

对于越来越多的网站被脱库，为了保证自家用户信息的安全性，开启两步验证成为了很多人的选择。

### 两步验证的原理

服务器首先要给每个用户生成一个 KEY，然后按照 [HOTP](http://tools.ietf.org/html/rfc4226) 或者 [TOTP](http://tools.ietf.org/html/rfc6238) 协议生成 CODE，根据协议不同，生成的 CODE 分别是基于计数的或者 基于时间的。

如果是 Time Based 的 CODE, 就会对时间要求比较高，由于短信通知的及时性不能保证，所以这种情况多采用 APP 方式。

对于 APP 方式，这里简单介绍一下 [Google Authenticator](https://en.wikipedia.org/wiki/Google_Authenticator) 

#### Google Authenticator

Google Authenticator 是实现了 TOTP 和 HOTP 两种方式，所以他可以生成基于时间和基于计数两种 CODE。 它支持两种方式，二维码输入或者手动输入。

它可以保存用户的账号和 KEY，然后根据协议实时生成最新的 CODE。

![Google Authenticator](/images/google_authenticator_snapshot.png)

### 两步验证的安全性

在整个两步验证的过程中，除了用户名和密码以外，还有两层防护措施：

1. KEY 的保密性
2. CODE 的时间限制或者次数限制

在实现中，要尽可能保证这两条均是分开通知，这样即可保证在某一部分被盗取也能保证用户信息的安全。

### 两步验证的实践

如果 Ruby 是一门强大的语言，活跃的社区肯定是功不可没。

关于两步验证，社区已经有多个实现。大家可以自己选用：

+ [Ruby 实现: rotp](https://github.com/mdp/rotp)
+ [Rails 实现: active_model_otp](https://github.com/heapsource/active_model_otp)
+ [Devise 集成: devise-two-factor](https://github.com/tinfoil/devise-two-factor)
+ [Devise 集成: two_factor_authentication](https://github.com/Houdini/two_factor_authentication)
+ [Devise 集成：devise-otp](https://github.com/wmlele/devise-otp)


enjoy!
