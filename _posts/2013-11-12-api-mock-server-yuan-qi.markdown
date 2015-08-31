---
layout: en_post
title: "api mock server"
date: 2013-11-12 08:53
comments: true
category: en
tags: api mock
---

I am happy to announce that [API Mock Server](https://github.com/zlx/API-mock-server) 1.2 has been released.

<!--more-->

### What is it?

The Project is used for mock RESTful API server.

You can create apis with json format response, and then you will receive the configured response when visit the api.

### when does it used?

When you development with mobile app, it's more and more pop to provide a API server for data process and storage.

But for most of the time, we must define and complete the API server development and then our mobile app. It's less agile and less flexible.

So the project appeared. It can deal with the situation: After we define our API data formate, we can mock the APIs, so mobile team and server team can work independently, just match the mock APIs.


### What does it work?

The Project is base on sinatra and mongodb as database.

I provide a api back management for create, edit, delete and show API, which mount to `/admin`

You can configure top_namespace for your api global namespace, for example: `api/v1` will make your api avaliable `<your-server-host>/api/v1/<your-created-api>`

And you may want to config admin username/password for authencation to your back management.

That's all features right now.

Welcome issue and contribution on [api-mock-server](https://github.com/zlx/API-mock-server)


enjoy!
