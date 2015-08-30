---
layout: post
title: "获取用户房间列表"
date: 2013-07-21 08:11
comments: true
category: tech
tags: ejabberd module muclist
---

最近在基于 [XMPP](http://xmpp.org/) 开发聊天系统，使用 [ejabberd](http://www.ejabberd.im/) 作为我们的服务器。由于业务需要，需要服务器提供一个接口，可以返回用户当前的房间列表。

<!--more-->

经过几天的研究，我发现可以利用 ejabberd 提供的 [IQ handlers](http://www.ejabberd.im/IQ+handlers) 方式，注册一个新的 IQ handler，用来处理这个请求。

首先我们需要创建一个模块，实现 gen_mod，

    -module(mod_user_muclist).
    
    -behaviour(gen_mod).

在这里，我们创建一个新的 Namespace 来定义这种请求，

    -define(NS_MUC_LIST, "http://jabber.org/protocol/muc#mylist").

按照 gen_mod 的要求，我们需要实现 start/2, stop/1 两个方法。ejabberd 启动时，会调用这个模块的 start/2 ，ejabberd 退出时会调用 stop/2，所以我们就在 start/2 里面注册我们的 IQ handler， 在 stop/1 里面取消注册。

    -export([start/2, stop/1, process_user_muclist_iq/3]).
    
    start(Host, Opt) -> 
        ?INFO_MSG("process_user_muclist_iq loading", []),
        IQDisc = gen_mod:get_opt(iqdisc, Opt, one_queue),
        mod_disco:register_feature(Host, ?NS_MUC_LIST),
        gen_iq_handler:add_iq_handler(ejabberd_local, Host, ?NS_MUC_LIST,
                      ?MODULE, process_user_muclist_iq, IQDisc).
     
    stop(Host) -> 
        ?INFO_MSG("stopping process_user_muclist_iq", []),
        mod_disco:unregister_feature(Host, ?NS_MUC_LIST),
        gen_iq_handler:remove_iq_handler(ejabberd_local, Host, ?NS_MUC_LIST).

注意: *除了要调用 gen_iq_handler:add_iq_handler 添加你的 IQ handler 以外，还要使用 mod_disco:register_feature 来告诉 mod_disco 模块你的需要开放 NS_MUC_LIST 的功能。*

按照 [XEP-0030](http://xmpp.org/extensions/xep-0030.html) 协议的一般规范，我们提供了 get 和 set 的两个方法，

    process_user_muclist_iq(From, To, #iq{xmlns = NS, id = ID, type = set, lang = Language, sub_el = SubEl} = IQ) ->
        {iq, ID, error, NS, Language, []};
    process_user_muclist_iq(From, To, #iq{xmlns = NS, id = ID, type = get, lang = Language, sub_el = SubEl} = IQ) ->
        ?INFO_MSG("From: ~s, To: ~s, ID: ~s, Element: ~s~n", [jlib:jid_to_string(From), jlib:jid_to_string(To), SubEl]),
        Res = IQ#iq{type = result, sub_el = [{xmlelement, "query", [{"xmlns", NS}], iq_user_muclist(From, To)}]},
        ejabberd_router:route(To, From, jlib:iq_to_xml(Res)).

这里我们不需要 set，所以就返回一个错误节。我们主要在 get 的方法里面构造结果，并调用 ejabberd_router:route 返回处理结果。这里我们主要使用了 iq_user_muclist/2 来构造结果。

    iq_user_muclist(From, To) ->
        MucRooms = mnesia:dirty_all_keys(muc_room),
        lists:map(fun({Name, Host}) ->
              {xmlelement, "item", [{"jid", jlib:jid_to_string({Name, Host, ""})}], []}
            end
            ,lists:filter(fun({Name, Host}) ->
                  is_user_in_room(From, Name, Host)
                end,
                MucRooms)).

首先从 muc_room 找出所有的纪录，然后调用 is_user_in_room/3 找出 From 这个用户所在的房间，最后将结果构造成结果的格式。

    is_user_in_room(From, Name, Host) ->
      [{muc_room, _, Opts}] = mnesia:dirty_read(muc_room, {Name, Host}),
      {affiliations, Affiliations} = lists:nth(25, Opts),
      lists:any(fun({
      {Name, Host, _}, _
      }) ->
            jlib:make_jid(Name, Host, "") == jlib:make_jid(From#jid.user, From#jid.server, "")
          end,
          Affiliations). 

is_user_in_room/3 这个方法首先根据 Name 和 Host 找出对应的房间纪录，然后找出这个房间的所有成员： Affiliations， 如果 From 在 Affiliations 里面，着返回 true， 否则返回 false。

至此，我们就可以编译这个模块，并将 beam 文件拷贝到 ejabberd 的 ebin 目录下，然后在配置文件中加入 `{mod_user_muclist, []}` 来启用这个模块。

### 请求格式

我们可以发送这样的请求：

    <iq type='get'
        from='romeo@montague.net/orchard'
        to='shakespeare.lit'
        id='muclist1'>
      <query xmlns='http://jabber.org/protocol/muc#mylist'/>
    </iq>

服务器会返回这样的结果：

    <iq type='result'
        from='shakespeare.lit'
        to='romeo@montague.net/orchard'
        id='muclist1'>
      <query xmlns='http://jabber.org/protocol/muc#mylist'>
        <item jid='room1@conference.shakespeare.lit'/>
        <item jid='room2@conference.shakespeare.lit'/>
        <item jid='room3@conference.shakespeare.lit'/>
        <item jid='room4@conference.shakespeare.lit'/>
      </query>
    </iq>


### 参考

+ [ejabberd modules](https://github.com/stockr-labs/ejabberd-modules)
+ [ejabberd IQ handler](http://www.ejabberd.im/IQ+handlers)

enjoy!
