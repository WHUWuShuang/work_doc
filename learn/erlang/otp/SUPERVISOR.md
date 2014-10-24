supervisor实际上是基于gen_server的系统进程，监控子进程的退出状态并设置一定的重启机制。

####INIT
![](http://www.hoterran.info/wp-content/uploads/2012/05/supervisor_v2-init.png)
在这个例子里Mod模块是一个sup程序,它的启动会调用supervisor:start_link,而start_link实际上调用的gen_server:start_link并存入Mod模块的名字和参数.

从前面的文章我们可以知道, spawn出来的gen进程会先调用supervisor:init函数. 接着把gen进程设置为系统进程, 这样就可以捕获子进程退出信号, 然后根据Args里的Mod模块名和参数,再次调用到Mod:init.

Mod的init函数返回的是一个{ok, SupFlags, StartSpec}的元组. SupFlags是supervisor管理的进程的启动策略和可重启的范围窗口，StartSpec是一个列表,保存多个子进程的MFA等信息.

接下来的init_state函数会把SupFlags, StartSpec存储在gen进程的State里, gen进程在整个生命周期一直围绕着这个State. 我们来看看State的结构,明白这个结构,supervisor就很好理解了.
```erlang
-record(state, {
    name,
    strategy,
    children = [],
    dynamics = ?DICT:new(),
    intensity,
    period,
    restarts = [],
    module,
    args
}).
```
strategy、intensity、period和SupFlags里信息一一对应，module、args就是这个supervisor的模块名字和参数。

额外的字段说明: restarts用于存储子进程的每个重启时间点, 这个后面会解释. dynmiacs用于当策略是simple_one_for_one策略时存储子进程的信息的字典,子进程的异常退出和重启都要用到到它；对于其它的策略,子进程信息存储在children里,它是一个child记录的列表.
```erlang
-record(child, {
    pid = undefined,  % pid is undefined when child is not running
    name,
    mfa,
    restart_type,
    shutdown,
    child_type,
    modules = []
}).
```
child记录和StartSpec里一一对应很好理解不解释了.

要注意对于非simple_one_for_one策略StartSpec里可以有多个子进程spec, 而如果是simple_one_for_one策略只能有一个进程的spec.下面的simple_one_for_one类策略的StartSpec保存函数可看到, 如果列表里的成员个数超过一个则报出bad_start_spec的错误.
```erlang
init_dynamic(State, [StartSpec]) ->
    case check_startspec([StartSpec]) of
    ....
init_dynamic(_State, StartSpec) ->
    {stop, {bad_start_spec, StartSpec}}.
```
存储好State信息后，gen进程根据策略的不同走到不同的分支, 如果策略是simple_one_for_one, 则在init的时候不启动任何子进程, 而由之后的start_child函数启动；对于其它的策略，则是遍历children列表把子程一一启动，并填充#child.pid字段. 等子进程都准备好了就发送ack通知调用进程，子进程都已经准备好了让我们开始吧，于是调用进程就成功退出。

####START_CHILD
![](http://www.hoterran.info/wp-content/uploads/2012/05/supervisor_v2-start_child.png)

对于非simple_one_for_one策略的子进程可以跳过这步，上面说过simple_one_for_one策略的子进程，需要显示的调用supervisor:start_child()来启动子进程.

start_child实际上是通过gen_server:call来通知gen进程启动一个子进程，然后gen进程回调调用handle_call函数。
```erlang
handle_call({start_child, EArgs}, _From, State) when ?is_simple(State) ->
    #child{mfa = {M, F, A}} = hd(State#state.children),
    Args = A ++ EArgs,
    case do_start_child_i(M, F, Args) of
...
```
gen进程接收到消息后, 会从State里获取需要启动的子进程的MFA, 最后调用apply(M,F,A)启动这个子进程。子进程的参数来自于ssupervisor:start_link和supervisor:start_child的两次参数组合, do_start_child_i(M, F, Args) 就是启动这个子进程的函数, 之后把{Pid, Args}信息再存储在#state.dynmiac里方便重启或者which_children展示子进程状态,.

update:

对于非simple_one_for_one策略也是可以动态添加子进程的.
```erlang
handle_call({start_child, ChildSpec}, _From, State) ->
    case check_childspec(ChildSpec) of
    {ok, Child} ->
        {Resp, NState} = handle_start_child(Child, State),
        {reply, Resp, NState};
    What ->
        {reply, {error, What}, State}
    end;
```
ChildSpec就是子进程的StartSpec, 那么如何调用?
```erlang
start_child(Args) ->
    Spec = {Args, {template_child, start_link, [Args]},
            permanent, brutal_kill, worker, [test3]},
    supervisor:start_child(?MODULE, Spec).
```
另外要注意对于start_child一次只能加一个子进程, 而不像init一次可以添加多个进程, 但这里添加的子进程信息非常的灵活, 太赞了.

####进程异常退出

由于gen进程是一个系统进程,所以子进程退出之际会向gen进程发送’EXIT’消息。根据gen_server的特点,这个消息会触发gen进程调度handle_info函数.
```erlang
handle_info({'EXIT', Pid, Reason}, State) ->
    case restart_child(Pid, Reason, State) of
        {ok, State1} -> {noreply, State1};
        {shutdown, State1} -> {stop, shutdown, State1}
    end;
```
于是gen进程就开始重启子进程.

####RESTART_CHILD
![](http://www.hoterran.info/wp-content/uploads/2012/05/supervisor_v2-restart_child.png)

重启进程会根据消息里Pid查找到进程的信息, 如前面所述, 对与simple_one_for_one策略的进程信息来自于从#state.dynmaics, 而其它的是从#state.children里读取MFA,RestartType等信息.

根据RestartType的不同决定是否要重启
```erlang
do_restart(permanent, Reason, Child, State) ->
    report_error(child_terminated, Reason, Child, State#state.name),
    restart(Child, State);
do_restart(_, normal, Child, State) ->
    NState = state_del_child(Child, State),
    {ok, NState};
do_restart(_, shutdown, Child, State) ->
    NState = state_del_child(Child, State),
    {ok, NState};
do_restart(transient, Reason, Child, State) ->
    report_error(child_terminated, Reason, Child, State#state.name),
    restart(Child, State);
do_restart(temporary, Reason, Child, State) ->
    report_error(child_terminated, Reason, Child, State#state.name),
    NState = state_del_child(Child, State),
    {ok, NState}.
```    
可以看到如果是permananet则永远重启；transient仅在子进程退出Reason非normal、shutdown才重启；而对于temprory状态则永远不重启，并删除State里的这个子进程的信息.

接着来到restart(Child, State)函数,这个函数首先通过add_restart函数记录一个重启时间点到#tates.restarts里，这样#state.restarts里保存着最近的重启时间点。计算当前时间的period时间之内的重启时间点的个数是否超过intensity, 如果超过则表示在period时间内子进程重启的次数超过maxintensity，是则放弃这次重启，否则继续重启来到restart函数.

restart函数有会根据#state.strategy来决定是重启单个进程还是所有进程，最后调用到do_start_child或者do_start_child_i函数，然后就是 apply(M,F,A)于是进程就重启了，然后再更新State里的信息等收尾工作。

####WHICH_CHILDREN

这个命令很有用,可以看到一个supervisor进程下有多少个子进程和他的状态,是在看代码的时候才发现的.

[原文链接](http://www.hoterran.info/otp-supervisor-sourcecode)
