gen_fsm和gen_server非常的类似, 在gen进程递归调用loop函数的过程中，除有StateData还额外有一个StateName的atom, 它决定了下次执行的函数. 另外一个不同之处是, gen_server程序是由调用进程向gen进程发送消息, 一种cs模式的调用关系，而gen_fsm程序中这个发送消息的通常都是gen进程本身. 

####INIT
![](http://www.hoterran.info/wp-content/uploads/2012/05/gen_fsm_v2-init.png)

初始化过程和gen_server很类似,  区别是init返回必须获得一个当前状态StateName, 这样才能继续接下来的事件处理.

如果在Mod:init返回的是{ok, State, Timeout}
```erlang
{ok, State, Timeout} ->
proc_lib:init_ack(Starter, {ok, self()}),
loop(Parent, Name, State, Mod, Timeout, Debug);
```
则在loop函数的阻塞是有超时的，不会永久阻塞等待消息。
```erlang
loop(Parent, Name, StateName, StateData, Mod, Time, Debug) ->
Msg = receive
Input ->
Input
after Time ->
{'$gen_event', timeout}
end,
decode_msg(Msg,Parent, Name, StateName, StateData, Mod, Time, Debug, false).
```
即便超时时间内gen进程未收到任何消息, 依然会触发StateName函数的执行, 其中Event为timeout.
```erlang
dispatch({'$gen_event', Event}, Mod, StateName, StateData) ->
Mod:StateName(Event, StateData);
```
对于gen_server也一样可以设置为接收到消息的超时, 而他触发的函数是hanle_info, 其中Info也为timeout.
```erlang
dispatch(Info, Mod, State) ->
Mod:handle_info(Info, State).
```
####SEND_EVENT
![](http://www.hoterran.info/wp-content/uploads/2012/05/gen_fsm_v2-send_event.png)

异步事件和gen_server:cast类似并不等待gen进程的回馈, gen进程接受到异步事件和,根据当前状态StateName和异步事件Event模式匹配找到需要执行的StateName函数.

####SYNC_SEND_EVENT
![](http://www.hoterran.info/wp-content/uploads/2012/05/gen_fsm_v2-sync_send_event.png)

同步事件类似与gen_server:call,调用进程发送事件之后会等待gen进程的消息回馈(wait_resp_mon). 而gen进程同样根据当前的状态和同步事件决定需要执行的函数.

和call一样sync类的消息可以选择是否需要回馈信息, 如果不回馈信息则wait_resp_mon会超时退出.

包括下面sync_send_all_state_event的函数都可以调用sync_send_event/3来设置超时参数来控制在wait_resp_mon函数处阻塞的时间.

####SEND_ALL_STATE_EVENT

以上两个事件处理函数需要根据当前状态和决定函数名, 而全状态事件仅仅根据事件来决定要执行的函数, 无论当前处于哪种状态，在统一的函数handle_event里执行.

####SYNC_SEND_ALL_STATE_EVENT
![](http://www.hoterran.info/wp-content/uploads/2012/05/gen_fsm_v2-sync_send_all_state_event.png)

同上,只是调用需要等到gen进程的消息回馈, 统一在handle_sync_event里处理.

####INFO

和gen_server一样,不同之处多返回一个StateName.

####TERMINATE

和gen_server一样.

[原文链接](http://www.hoterran.info/otp-gen_fsm-sourcecode)