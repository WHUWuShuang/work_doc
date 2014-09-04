>阅读OTP源码可以帮助你写出更好、更健壮的erlang程序.下面一系列文章就gen_server、gen_fsm、supervisor的源码进行分析, 从erlang级别解释其工作原理, 所有的完整流程图在这里, 第一次写erlang方面博文有错误请帮忙指出.

>为什么从gen_server它开始, 因为gen_fsm和它很类似, 而supervsisor本身是一个gen_server.

***
###**INIT**
![](http://www.hoterran.info/wp-content/uploads/2012/05/gen_server-init1.png)


图示为一个叫Mod的模块, 它是一个gen_server程序, 绿色方格为调用进程（客户进程）, 黄色方格为spawn出的gen进程（服务进程）. 不同的泳道表示函数所隶属的模块, 通过这个图可以清晰的看出各个模块至之间的相互调用, 图是使用gliffy所画。

从左上角的start(Args)开始，gen_server的程序Mod的开始函数都会调用gen_server:start(或者start_link)来创建一个服务进程，gen_server:start内部实际上调用的是gen模块。

gen模块是很多behaviour的基础,对于gen进程的启动和同步操作进行了封装.gen模块使用proc_lib:start来更加安全的spawn出gen进程，然后阻塞在sync_wait这个函数里,等待gen进程的回应.

spawn出的gen进程会执行到Mod模块的init函数。然后会把{ok, Pid}发送给调用进程告知gen进程已经完成准备工作，然后就进入了自己的循环函数loop中，等待调用进程的下次消息。

调用进程接收到{ok，Pid}之后从阻塞里退出并返回。

整个过程简单，但是要清楚哪个函数是被调用进程或者gen进程所执行。

另外要注意start函数的参数是Args，而init函数的入参是[Args]，这个很容易出错的地方。

可以调用start/4或者start_link/4给启动的gen进程命名,函数name_register实际调用register函数.

    init_it(GenMod, Starter, Parent, Name, Mod, Args, Options) ->  
     	case name_register(Name) of  
     	true ->  
     		init_it2(GenMod, Starter, Parent, Name, Mod, Args, Options);
     		....
###**CAST**


![](http://www.hoterran.info/wp-content/uploads/2012/05/gen_server-cast1.png)
调用进程向gen进程发送cast消息，消息发送之后调用进程并不等待gen进程的消息回馈.

gen进程从loop函数处接受到Request消息，模式匹配后一路执行到Mod:handle_cast函数，处理消息之后，gen进程继续递归执行loop函数等待后续的新消息. 注意handle_cast不能返回{reply,…}，否则gen进程会报错退出。

###**CALL**
![](http://www.hoterran.info/wp-content/uploads/2012/05/gen_server-call1.png)


调用进程向gen进程发送call消息，和cast不同，调用进程会阻塞在wait_resp_mon函数里等待gen进程的回馈。收到消息后，gen进程会执行Mod:handle_call函数，并把执行的结果Reply直接发送给调用进程，然后自己再次进入循环等待新的消息。

	wait_resp_mon(Node, Mref, Timeout) ->
		receive
		    {Mref, Reply} ->
		        erlang:demonitor(Mref, [flush]),
		        {ok, Reply};
		    {'DOWN', Mref, _, _, noconnection} ->
		        exit({nodedown, Node});
		    {'DOWN', Mref, _, _, Reason} ->
		        exit(Reason)
		after Timeout ->
	        erlang:demonitor(Mref),
	        receive
	        	{'DOWN', Mref, _, _, _} -> true
	        	after 0 -> true
	        end,
	        exit(timeout)
	    end.
默认情况下handle_call的返回是{reply,….}. 而调用进程阻塞在wait_resp_mon的默认超时时间是5s(-define(default_timeout, 5000)).

在spec里看到handle_call的返回值可以是{noreply,…},  或者gen进程在处理其它事情而达到超时时间, 则调用进程会异常退出, 你也可以在调用gen_server:call/3来设置一个call命令的超时时间.

对于call和cast的命令既可以使用Name也可以使用Pid,  gen内部会进行Name到Pid的转化。

	call(Name, Label, Request, Timeout)
		when is_atom(Name), Timeout =:= infinity;
			is_atom(Name), is_integer(Timeout), Timeout >= 0 ->
		case whereis(Name) of
			Pid when is_pid(Pid) ->
				do_call(Pid, Label, Request, Timeout);
 

###**INFO**
![](http://www.hoterran.info/wp-content/uploads/2012/05/gen_server-info1.png)


可以给gen进程发送任何格式的信息，这类信息没有gen标签，gen进程接收到这类消息会调用gen_server:handle_info来处理消息，处理过程与cast消息一样都不能反馈结果给调用进程，所以如果handle_info返回{reply，…}也会导致gen进程报错退出。

###**TERMINATE**
![](http://www.hoterran.info/wp-content/uploads/2012/05/gen_server-terminate1.png)


 

在handle_*等回调函数处都可以返回{stop,…} 这样使得gen进程执行Mod:terminate, 进行进程退出前的收尾工作, 然后回到主循环gen进程再退出.

[原文链接](http://www.hoterran.info/otp-gen_server-sourcecode)
