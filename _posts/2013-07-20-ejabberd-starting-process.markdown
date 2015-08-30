---
layout: post
title: "Ejabberd 启动过程"
date: 2013-07-20 14:23
comments: true
category: tech
tags: Ejabberd init modules
---

最近研究 [Ejabberd](http://www.ejabberd.im/) 我准备将我研究的一些心得写成博客，供有需要的人借鉴。由于本人水平有限，如有错误，欢迎指正！

今天我们讲一下 Ejabberd 的启动过程。

<!--more-->

Ejabberd 是基于 [Otp](http://en.wikipedia.org/wiki/OTP) 框架开发的。 Otp 框架是一个流行的 [erlang](http://www.erlang.org/) 框架，它封装了繁琐的底层实现，对外主要暴露了四个接口，来实现多并发编程。这四个接口分别是：gen_server、 gen_fsm、 gen_event 和 supervisor。其中 gen_server 是 client-server 关系中的 server 实现；gen_fsm 是有限状态自动机的实现；gen_event 是事件处理机制的实现以及 supervisor 是[监督者](http://www.erlang.org/doc/design_principles/des_princ.html)的实现。

一个标准的 Otp 应用程序都会有一个 <yourname>.app 文件，其中描述了应用程序的基本信息，其主要结构为：

*{application, ApplicationName, Properties}.*

其中 Properties 是一个包含很多键值对的列表：

*{description, "Some description of your application"}*

描述了这个应用程序的描述信息；

*{vsn, "1.2.3"}*

描述了这个应用程序的版本；

*{modules, ModuleList}*

描述了这个应用程序所包含的模块；

*{registered, AtomList}*

描述了这个应用程序注册的名称；

*{env, [{Key, Val}]}*

描述了这个应用程序需要的环境配置；

*{maxT, Milliseconds}* 

描述了这个应用程序能运行的最长时间；

*{applications, AtomList}*

描述了这个应用程序依赖的应用程序；

*{mod, {CallbackMod, Args}}*

描述了这个应用程序的回调模块，应用程序启动后会调用 *CallbackMod:start(normal, Args)* ；


## Ejabberd 的应用程序描述文件（以 2.1.x 为范例）。

    {application,ejabberd,
                 [{description,"ejabberd"},
                  {vsn,"community"},
                  {modules,['ELDAPv3','XmppAddr',acl,adhoc,configure,cyrsasl,
                            ...
                            scram,shaper,str,translate,treap,win32_dns]},
                  {registered,[]},
                  {applications,[kernel,stdlib]},
                  {env,[]},
                  {mod,{ejabberd_app,[]}}]}.

可以看出这个应用程序的名字叫 ejabberd， 描述为 “ejabberd”，版本为 “community”，模块一大堆，依赖 kernel 和 stdlib ，回调模块为 ejabberd_app 。

由此，我们可以知道 Ejabberd 的程序入口为 *ejabberd_app:start(normal, Args)*;

    start(normal, _Args) ->
        ejabberd_loglevel:set(4),
        write_pid_file(),
        application:start(sasl),
        randoms:start(),
        db_init(),
        sha:start(),
        stringprep_sup:start_link(),
        xml:start(),
        start(),
        translate:start(),
        acl:start(),
        ejabberd_ctl:init(),
        ejabberd_commands:init(),
        ejabberd_admin:start(),
        gen_mod:start(),
        ejabberd_config:start(),
        ejabberd_check:config(),
        connect_nodes(),
        %% Loading ASN.1 driver explicitly to avoid races in LDAP
        catch asn1rt:load_driver(),
        Sup = ejabberd_sup:start_link(),
        ejabberd_rdbms:start(),
        ejabberd_auth:start(),
        cyrsasl:start(),
        % Profiling
        %ejabberd_debug:eprof_start(),
        %ejabberd_debug:fprof_start(),
        maybe_add_nameservers(),
        start_modules(),
        ejabberd_listener:start_listeners(),
        ?INFO_MSG("ejabberd ~s is started in the node ~p", [?VERSION, node()]),
        Sup;
    start(_, _) ->
        {error, badarg}.
    
### *ejabberd_loglevel:set(4)* 

设定了日志的默认等级为 4；

### *write_pid_file()* 

将 pid 写入文件；

### *application:start(sasl)* 

启动 sasl, application:start(App) 这样会保证 App 在应用程序启动前启动，否则会抛出错误 {error, 
{not_started, App}}；

### *randoms:start()* 

启动 randoms 模块

    -module(randoms).
    start() ->
        register(random_generator, spawn(randoms, init, [])).
    init() ->
        {A1, A2, A3} = now(),
        random:seed(A1,A2,A3),
        loop().
    loop() ->
        receive
        {From, get_random, N} ->
            From ! {random, random:uniform(N)},
            loop();
        _ ->
            loop()
        end.

randoms 模块很简单，*register(random_generator, spawn(randoms, init, [])).* 注册一个叫 random_generator 的进程，这个进程接受消息并以当前时间为种子，产生随机数。

### *db_init()* 

初始化数据库

    db_init() ->
        case mnesia:system_info(extra_db_nodes) of
        [] ->
            mnesia:create_schema([node()]);
        _ ->
            ok
        end,
        application:start(mnesia, permanent),
        mnesia:wait_for_tables(mnesia:system_info(local_tables), infinity).

它首先检查 mnesia 数据库有没有创建好，没有就基于 [node()] 重新创建，然后启动 mnesia 并等待表创建完成。

### *sha:start()* 

启动 sha 模块

    -define(DRIVER, sha_drv).
    start() ->
        crypto:start(),
        Res = case erl_ddll:load_driver(ejabberd:get_so_path(), ?DRIVER) of
              ok -> ok;
              {error, already_loaded} -> ok;
              Err -> Err
          end,
        case Res of
        ok ->
            Port = open_port({spawn, atom_to_list(?DRIVER)}, [binary]),
            register(?DRIVER, Port);
        {error, Reason} ->
            ?CRITICAL_MSG("unable to load driver '~s': ~s",
                  [driver_path(), erl_ddll:format_error(Reason)])
        end.

他首先启动 crypto 服务器，然后调用 erl_ddll 加载 sha_drv 驱动，最后将为它注册一个进程来处理请求。

### *stringprep_sup:start_link()* 

启动 stringprep_sup 模块

    -module(stringprep_sup).
    -behaviour(supervisor).
    -define(SERVER, ?MODULE).
    start_link() ->
        supervisor:start_link({local, ?SERVER}, ?MODULE, []).
    init([]) ->
        StringPrep = {stringprep,
              {stringprep, start_link, []},
              permanent,
              brutal_kill,
              worker,
              [stringprep]},
        {ok,{
             {one_for_all,10,1},
             [StringPrep]
            }}.

这个模块实现了监督者接口，并把自身注册为一个监督者，并在 init/1 里面声明了重启策略，最大重启次数和子进程。

    -module(stringprep).
    -behaviour(gen_server).
    -define(STRINGPREP_PORT, stringprep_port).
    start_link() ->
        gen_server:start_link({local, ?MODULE}, ?MODULE, [], []).
    init([]) ->
        case erl_ddll:load_driver(ejabberd:get_so_path(), stringprep_drv) of
        ok -> ok;
        {error, already_loaded} -> ok
        end,
        Port = open_port({spawn, "stringprep_drv"}, []),
        register(?STRINGPREP_PORT, Port),
        {ok, Port}.

stringprep 实现了 gen_server 接口，同 sha 模块一样，它也加载了自己的驱动并注册了一个进程来处理请求。

### *xml:start()* 

启动 xml 模块

    %% Replace element_to_binary/1 with NIF
    %% Can be choosen with ./configure --enable-nif
    -ifdef(NIF).
    start() ->
        SOPath = filename:join(ejabberd:get_so_path(), "xml"),
        case catch erlang:load_nif(SOPath, 0) of
        ok ->
            ok;
        Err ->
            ?WARNING_MSG("unable to load xml NIF: ~p", [Err])
        end.
    -else.
    start() ->
        ok.
    -endif.

它仅仅根据配置来决定加载 xml 库文件。

### *start()*  

这里派生出一个子进程

    start() ->
        spawn_link(?MODULE, init, []).
    init() ->
        register(ejabberd, self()),
        %erlang:system_flag(fullsweep_after, 0),
        %error_logger:logfile({open, ?LOG_PATH}),
        LogPath = get_log_path(),
        error_logger:add_report_handler(ejabberd_logger_h, LogPath),
        erl_ddll:load_driver(ejabberd:get_so_path(), tls_drv),
        case erl_ddll:load_driver(ejabberd:get_so_path(), expat_erl) of
        ok -> ok;
        {error, already_loaded} -> ok
        end,
        Port = open_port({spawn, "expat_erl"}, [binary]),
        loop(Port).

可以看出，这里将派生出一个自身的子进程，在这个子进程里面调用 error_logger:add_report_handler 添加日志的处理器，然后加载 tls_drv 和 expat_erl 驱动， 然后在 expat_erl 的端口等待请求。

### *translate:start()* 

启动 translate 模块

    -module(translate).
    start() ->
        ets:new(translations, [named_table, public]),
        Dir = 
        case os:getenv("EJABBERD_MSGS_PATH") of
            false ->
            case code:priv_dir(ejabberd) of
                {error, _} ->
                ?MSGS_DIR;
                Path ->
                filename:join([Path, "msgs"])
            end;
            Path ->
            Path
        end,
        load_dir(Dir),

它在 ets 里面新建了一个叫做 translations 的表，并将 ejabber_msg 路径下的文件内容导入 translations 表。

### *acl:start()* 

启动 acl 模块

    -module(acl).
    -record(acl, {aclname, aclspec}).
    start() ->
        mnesia:create_table(acl,
                [{disc_copies, [node()]},
                 {type, bag},
                 {attributes, record_info(fields, acl)}]),
        mnesia:add_table_copy(acl, node(), ram_copies),
        ok.

它主要在 mnesia 里面创建了一个 acl 的表；

### *ejabberd_ctl:init()* 

初始化 ejabberd_ctl 模块，它在 ets 上面创建了 ejabberd_ctl_cmds 和 ejabberd_ctl_host_cmds 两张表。

### *ejabberd_commands:init()* 

初始化 ejabberd_commands 模块， 它创建了 ejabberd_commands 表。

### *ejabberd_admin:start()* 

启动了 ejabberd_admin 模块

    start() ->
        ejabberd_commands:register_commands(commands()).
    commands() ->
        [
         %% The commands status, stop and restart are implemented also in ejabberd_ctl
         %% They are defined here so that other interfaces can use them too
         #ejabberd_commands{name = status, tags = [server],
                desc = "Get status of the ejabberd server",
                module = ?MODULE, function = status,
                args = [], result = {res, restuple}},
         #ejabberd_commands{name = stop, tags = [server],
                desc = "Stop ejabberd gracefully",
                module = init, function = stop,
                args = [], result = {res, rescode}},
        ...
         #ejabberd_commands{name = load, tags = [mnesia],
                desc = "Restore the database from text file",
                module = ?MODULE, function = load_mnesia,
                args = [{file, string}], result = {res, restuple}},
         #ejabberd_commands{name = install_fallback, tags = [mnesia],
                desc = "Install the database from a fallback file",
                module = ?MODULE, function = install_fallback_mnesia,
                args = [{file, string}], result = {res, restuple}}
        ].

它调用 ejabberd_commands 注册了许多命令。

    register_commands(Commands) ->
        lists:foreach(
          fun(Command) ->
              case ets:insert_new(ejabberd_commands, Command) of
              true ->
                  ok;
              false ->
                  ?DEBUG("This command is already defined:~n~p", [Command])
              end
          end,
          Commands).

register_commands 将注册的命令存入 ejabberd_commands 的表。

### *gen_mod:start()* 

启动 gen_mod 模块， 它仅仅在 ets 里面新建了一个 ejabberd_modules 的表。

### *ejabberd_config:start()* 

启动 ejabberd_config 模块

    start() ->
        mnesia:create_table(config,
                [{disc_copies, [node()]},
                 {attributes, record_info(fields, config)}]),
        mnesia:add_table_copy(config, node(), ram_copies),
        mnesia:create_table(local_config,
                [{disc_copies, [node()]},
                 {local_content, true},
                 {attributes, record_info(fields, local_config)}]),
        mnesia:add_table_copy(local_config, node(), ram_copies),
        Config = get_ejabberd_config_path(),
        load_file(Config),
        %% This start time is used by mod_last:
        add_local_option(node_start, now()),
        ok.

它在 mnesia 上面创建了 config 和 local_config 两张表，并读入配置文件 ejabberd.cfg 。 

### *ejabberd_check:config()* 

检查配置的一致性。

### *connect_nodes()*

    connect_nodes() ->
        case ejabberd_config:get_local_option(cluster_nodes) of
        undefined ->
            ok;
        Nodes when is_list(Nodes) ->
            lists:foreach(fun(Node) ->
                      net_kernel:connect_node(Node)
                  end, Nodes)
        end.

它调用 net_kernel 连接上配置的每个节点。

### *asn1rt:load_driver()* 

加载 ASN.1 驱动来避免 LDAP 中出现[竞争]()

### *ejabberd_sup:start_link()* 

启动 ejabberd_sup 模块

    start_link() ->
        supervisor:start_link({local, ?MODULE}, ?MODULE, []).
    init([]) ->
        Hooks =
        {ejabberd_hooks,
         {ejabberd_hooks, start_link, []},
         permanent,
         brutal_kill,
         worker,
         [ejabberd_hooks]},
        ...
        CacheTabSupervisor =
        {cache_tab_sup,
         {cache_tab_sup, start_link, []},
         permanent,
         infinity,
         supervisor,
         [cache_tab_sup]},
        {ok, {
          {one_for_one, 10, 1},
          [Hooks,
           NodeGroups,
           SystemMonitor,
           Router,
           SM,
           S2S,
           Local,
           Captcha,
           ReceiverSupervisor,
           C2SSupervisor,
           S2SInSupervisor,
           S2SOutSupervisor,
           ServiceSupervisor,
           HTTPSupervisor,
           HTTPPollSupervisor,
           IQSupervisor,
           STUNSupervisor,
           FrontendSocketSupervisor,
           CacheTabSupervisor,
           Listener]}}.

它注册自身为 ejabberd 的监督者，并启动 Hooks,
       NodeGroups,
       SystemMonitor,
       Router,
       SM,
       S2S,
       Local,
       Captcha,
       ReceiverSupervisor,
       C2SSupervisor,
       S2SInSupervisor,
       S2SOutSupervisor,
       ServiceSupervisor,
       HTTPSupervisor,
       HTTPPollSupervisor,
       IQSupervisor,
       STUNSupervisor,
       FrontendSocketSupervisor,
       CacheTabSupervisor,
       Listener
子进程。

其中 Hooks 启动 ejabberd_hooks 模块，并创建 hooks 表；

Router 启动 ejabberd_router 模块，

    init([]) ->
        update_tables(),
        mnesia:create_table(route,
                [{ram_copies, [node()]},
                 {type, bag},
                 {attributes,
                  record_info(fields, route)}]),
        mnesia:add_table_copy(route, node(), ram_copies),
        mnesia:subscribe({table, route, simple}),
        lists:foreach(
          fun(Pid) ->
              erlang:monitor(process, Pid)
          end,
          mnesia:dirty_select(route, [{
                                       {route, '_', '$1', '_'},
                                       [],
                                       ['$1']
                                      }])),
        {ok, #state{}}.

它首先更新表，删除掉过期的纪录，然后创建一个 route 的表，并根据路由纪录监控每个对应的进程。

SM 启动 ejabberd_sm 模块，

    init([]) ->
        update_tables(),
        mnesia:create_table(session,
                [{ram_copies, [node()]},
                 {attributes, record_info(fields, session)}]),
        mnesia:create_table(session_counter,
                [{ram_copies, [node()]},
                 {attributes, record_info(fields, session_counter)}]),
        mnesia:add_table_index(session, usr),
        mnesia:add_table_index(session, us),
        mnesia:add_table_copy(session, node(), ram_copies),
        mnesia:add_table_copy(session_counter, node(), ram_copies),
        mnesia:subscribe(system),
        ets:new(sm_iqtable, [named_table]),
        lists:foreach(
          fun(Host) ->
              ejabberd_hooks:add(roster_in_subscription, Host,
                     ejabberd_sm, check_in_subscription, 20),
              ejabberd_hooks:add(offline_message_hook, Host,
                     ejabberd_sm, bounce_offline_message, 100),
              ejabberd_hooks:add(remove_user, Host,
                     ejabberd_sm, disconnect_removed_user, 100)
          end, ?MYHOSTS),
        ejabberd_commands:register_commands(commands()),
        {ok, #state{}}.
    commands() ->
        [
         #ejabberd_commands{name = connected_users,
                   tags = [session],
                   desc = "List all established sessions",
                   module = ?MODULE, function = connected_users,
                   args = [],
                   result = {connected_users, {list, {sessions, string}}}},
         #ejabberd_commands{name = connected_users_number,
                   tags = [session, stats],
                   desc = "Get the number of established sessions",
                   module = ?MODULE, function = connected_users_number,
                   args = [],
                   result = {num_sessions, integer}},
         #ejabberd_commands{name = user_resources,
                   tags = [session],
                   desc = "List user's connected resources",
                   module = ?MODULE, function = user_resources,
                   args = [{user, string}, {host, string}],
                   result = {resources, {list, {resource, string}}}}
        ].    

他同样先更新表，然后创建了 session 和 session_counter 的表，还在 ets 上面创建了一个 sm_iqtable 的表，然后对在每个 host 上添加了 roster_in_subscription， offline_message_hook， remove_user 三个 hook，最后还调用 ejabberd_commands 注册了 connected_users， connected_users_number， user_resources 三个命令。

S2S 则启动了 ejabberd_s2s 模块

    init([]) ->
        update_tables(),
        mnesia:create_table(s2s, [{ram_copies, [node()]}, {type, bag},
                      {attributes, record_info(fields, s2s)}]),
        mnesia:add_table_copy(s2s, node(), ram_copies),
        mnesia:subscribe(system),
        ejabberd_commands:register_commands(commands()),
        mnesia:create_table(temporarily_blocked, [{ram_copies, [node()]}, {attributes, record_info(fields, temporarily_blocked)}]),
        {ok, #state{}}.
    commands() ->
        [
         #ejabberd_commands{name = incoming_s2s_number,
                   tags = [stats, s2s],
                   desc = "Number of incoming s2s connections on the node",
                   module = ?MODULE, function = incoming_s2s_number,
                   args = [],
                   result = {s2s_incoming, integer}},
         #ejabberd_commands{name = outgoing_s2s_number,
                   tags = [stats, s2s],
                   desc = "Number of outgoing s2s connections on the node",
                   module = ?MODULE, function = outgoing_s2s_number,
                   args = [],
                   result = {s2s_outgoing, integer}}
        ].    

它更新完表后，创建了 s2s 和 temporarily_blocked 的表，并注册了 incoming_s2s_number， outgoing_s2s_number 两个命令。

Local 启动了 ejabberd_local 模块

    -define(IQTABLE, local_iqtable).
    init([]) ->
        lists:foreach(
          fun(Host) ->
              ejabberd_router:register_route(Host, {apply, ?MODULE, route}),
              ejabberd_hooks:add(local_send_to_resource_hook, Host,
                     ?MODULE, bounce_resource_packet, 100)
          end, ?MYHOSTS),
        catch ets:new(?IQTABLE, [named_table, public]),
        update_table(),
        mnesia:create_table(iq_response,
                [{ram_copies, [node()]},
                 {attributes, record_info(fields, iq_response)}]),
        mnesia:add_table_copy(iq_response, node(), ram_copies),
        {ok, #state{}}.
   
它首先对每个 host 注册了一个路由，并添加了一个 local_send_to_resource_hook 的 hook，然后在 ets 上面创建了一个 local_iqtable 的表，最后还创建了一个 iq_response 的表。

Listener 启动了 ejabberd_listener 模块，它注册自身为监督者，并在 ets 上面新建了一张 listen_sockets 的表，并将配置的端口绑定信息存入表。

ReceiverSupervisor, C2SSupervisor, S2SInSupervisor, S2SOutSupervisor, ServiceSupervisor, HTTPSupervisor, HTTPPollSupervisor, FrontendSocketSupervisor, IQSupervisor, STUNSupervisor, CacheTabSupervisor 则分别启动了对应的模块，注册为监督者，监督各自的子进程。   

### *ejabberd_rdbms:start()* 

    start() ->
        %% Check if ejabberd has been compiled with ODBC
        case catch ejabberd_odbc_sup:module_info() of
        {'EXIT',{undef,_}} ->
            ?INFO_MSG("ejabberd has not been compiled with relational database support. Skipping database startup.", []);
        _ ->
            %% If compiled with ODBC, start ODBC on the needed host
            start_hosts()
        end.

它检查配置中是否有 odbc 的模块，如果有就开启对应的 host 的 odbc 子进程。

### *ejabberd_auth:start()* 

启动 ejabberd_auth 模块

    start() ->
        lists:foreach(
          fun(Host) ->
              lists:foreach(
            fun(M) ->
                M:start(Host)
            end, auth_modules(Host))
          end, ?MYHOSTS).

针对每个 host ，启动对应的验证模块。

### *cyrsasl:start()* 

启动 cyrsasl 模块

    start() ->
        ets:new(sasl_mechanism, [named_table,
                     public,
                     {keypos, #sasl_mechanism.mechanism}]),
        cyrsasl_plain:start([]),
        cyrsasl_digest:start([]),
        cyrsasl_scram:start([]),
        cyrsasl_anonymous:start([]),
        ok.

它创建了 sasl_mechanism 表，并启动了 cyrsasl_plain， cyrsasl_digest， cyrsasl_scram 和 cyrsasl_anonymous 模块。

### *maybe_add_nameservers()* 

它在 win32 系统下添加了 DNS 解析器。

### *start_modules()* 

    start_modules() ->
        lists:foreach(
          fun(Host) ->
              case ejabberd_config:get_local_option({modules, Host}) of
              undefined ->
                  ok;
              Modules ->
                  lists:foreach(
                fun({Module, Args}) ->
                    gen_mod:start_module(Host, Module, Args)
                end, Modules)
              end
          end, ?MYHOSTS).

它调用了 gen_mode 的 start_module 分别启动了每个 host 上的各个模块;

### *ejabberd_listener:start_listeners()* 

调用 ejabberd_listener:start_listeners 开始监听端口。

### *Sup* 

最后把 ejabberd_sup:start_link 返回地 pid 作为回调de返回值。

至此，启动过程结束，进程退出，后台进程监听着各个端口，监督者监督着各个子进程。整个系统开始有条不紊地工作。


参考资料：

- [otp 设计原则](http://www.cnblogs.com/me-sa/archive/2011/11/20/erlang0015.html)
- [otp 应用程序启动](http://www.cnblogs.com/me-sa/archive/2011/12/27/erlang0025.html)
- [learn some erlang](http://learnyousomeerlang.com/building-applications-with-otp)
- [Ejabberd 源码](https://github.com/processone/ejabberd)


enjoy!
