如果你需要赶上我们关于Promise的进度，可以看看这个系列前两篇文章[深入理解Promise五部曲--1.异步问题](https://github.com/WHUWuShuang/work_doc/blob/master/learn/js/Promises_The%20Trust%20Problem(part1).md)和[深入理解Promise五部曲--2.控制权转移问题](https://github.com/WHUWuShuang/work_doc/blob/master/learn/js/Promises_The%20Trust%20Problem(part2).md)。

Promise状态 == 信任
-----
在前面，我们说明了几个关于Promises如何工作的要点，这些要点是我们之所以可以信任promise机制作为控制转移的一种解决方案的基础。
这些要点直接来自Promises/A+规范。任何本地实现或者polyfill或者库都必须通过一个全面严格的测试来确定是否符合规范。
对于promises可靠性是最基本的，因为如果没有可靠性，那么你就跟使用普通的回调一样了。你必须谨慎地编写那些涉及到异步调用第三方库的代码。你必须自己来解决状态跟踪的问题然后确保第三方库不会出问题。
如果没有可靠的promises你自己可以完成异步任务吗？当然可以。但是问题是，你自己无法处理得很完美，你得把很多额外的变量加到你的代码中并且你会产生一个未来的维护风险，代码会变得很难维护。
**Promises是被设计用来规范和集中这种逻辑的**。你可以使用一个规范的promise系统而不用担心可靠性问题，因为它会按照Promises机制来执行。

可依赖吗？
-----
在理论上这个可靠性保证合同听起来很棒。但是在JavaScript中真的有可能有这么一个机制吗？

####可靠性

在我开始说这个问题之前，我们首先排除一些JS代码中的可靠性问题：

1.  我们这里的讨论跟密码/加密中的“私有性”和“安全”无关。
1.  和JS代码可以被用户通过查看源码看到无关。
1.  和一个黑客可以侵入你的服务器来发送一些恶意代码或者通过中间人攻击来劫持浏览器和服务器之间的连接来实现同样的目的或者甚至在运行时使用XSS漏洞来注入恶意代码无关。
1.  同时，也和恶意代码一旦存在你的页面就可以理论上修改JavaScript运行时功能(比如通过修改Object.prototype或者Function.prototype)来破坏你的程序这个事实无关。
1.  相似的，和一些粗心的代码可能会意外地通过非标准的方式来修改标准JS函数无关。
1.  最后，和如果你页面中依赖于第三方库那么他们的服务器，连接和代码也会出现上面所说的漏洞无关。

现在我可以继续了，但是我认为你已经找到关键点了。我们在通过一个假设来缩小我们的讨论范围：当所有的代码以及主机环境都在一种预期的安全的状态中时，你的程序会如何执行？
这并不是说我们使用Promise所做的事情对上面这些问题没有帮助。这仅仅是由于这些问题在一个更高的层面上---这些问题远离了编写API和模式，这些问题留给专家来讨论。

####在Promise状态下的可靠性

我们看看下面这个例子：
```js
var myPromise = {
    state: {
        status: 1,
        value: "Hello World"
    },
    then: function(success,failure) {
        // implement something like a thenable's behavior
    }
};
```
我可以新建一个像这样的对象，然后在平时使用它并且说我在用promises。实际是，我可以再完善一下使它可以通过整个[Promises/A+ 测试网站][promises test]的测试。

####但是我真的是使用了Promises吗？

你如何回答这个问题比你意识到的更重要。在很多开发者社区中很多人的回答是，是的。
我很确定的说，不是！
为什么？如果你通过了promises测试网站，那么它就是一个promise 了，不是吗？而且，它在所有情况下都按照规范来执行，不是吗？

####不是

promises的精髓远不是规范说的那么简单，是**可靠性**。
可靠性是一个promise就是一个状态(状态会从"pending"转变成"resolved"或者"rejected"其中一个)的容器，这些状态会附带一个结果值(成功信息或者错误信息)。可靠性是一旦一个promise的状态变为"resolved"或者"rejected"，那么就不能改变也不会改变。可靠性就是完成的promise是不可变的。
但是promises的精髓还有一些更深层次的东西，这些是无法通过阅读规范看出来的：改变一个promise状态和设置它的完成值的能力只存在于原始的promise的实现。也就是说这个能力的实现掌握在开发者手里。
规范的早期版本中，把resolve/reject的功能分离出来放在一个对象中，叫做**Deferred**.
把这想成一个对象对：在创建的时候，我们创建一个promise和一个deferred，deferred可以resolve这个promise。
重要的是，这两个可以被分开，一部分代码可以resolve/reject一个promise而另外一部分只能监听这个变化然后做出回应。
规范的后续版本中简化了promises，通过删除deferred对象，取而代之的是简单的暴露出原来属于deferred的resolve()和reject()方法。
```js
var p = new Promise( function(resolve,reject){
    // I have `resolve()` and `reject()` from the
    // hidden `deferred`, and I **alone** control
    // the state of the promise.
} );

// now, I can pass around `p` freely, and it can't
// be changed by anyone else but the creator.
```
看看之前的那个`myPromise`对象。你注意到了什么吗？
```js
var myPromise = {
    state: {
        status: 1,
        value: "Hello World"
    },
    then: function(success,failure) {
        // implement something like a thenable's behavior
    }
};
```
如果你到处传递m`yPromise`，然后不管恶意代码还是意外的代码都可以改变`myPromise.state.status`或者`myPromise.state.value`属性，我们是不是开了一个很大的后门，失去了Promises的可靠性。
当然，答案是肯定的。把状态暴露给方法使得这不是一个真正的promise。因为现在promise的保证已经完全不可靠了。
如果你从一个第三方库中得到了一个这样的对象，你不会信任它的，不是吗？更重要的，如果你把这个对象传递给其他第三方库，你肯定不会相信只有原始的创建者才能修改它，不是吗？
当然不会相信。那就太天真了。
你看，使用promises是基于可靠性的。然后可靠性是基于promise的状态是与外部影响隔离的，只有创建者能改变。注意到我并没有说状态必须是私有的，只要它不会被外界改变就可以。
如果没有promise的对象不会被除了创建者改变的可靠性，那么promise就几乎失去了它的意义。

错误的可靠性？
-----------
注意，这正是事情变得模糊的地方，是不可忽视的事实。
大多数为了在旧的JS环境下能够支持promise的polyfill会把状态通过可变的方式暴露出来。
Ouch!!!
在这方面，我的ES6 Promise polyfill"[Native Promise Only](https://github.com/getify/native-promise-only)"没有把state暴露出来。据我所知，这是唯一一个没有把promise状态暴露出来的polyfill。
为什么？因为我不仅仅关心Promise规范，我更在意Promises的精髓。

####Tradeoffs

但是究竟为什么所有这些高度可信的Promise polyfill和库会忘了promise中这么重要的东西呢？
因为在原生Javascript有一些限制，这是一些内置机制不需要遵循的。
简单的说，即将到来的ES6标准指出`Promise`是一个“class”，所以作为一个“class”，promise必须可以被子类化。
换句话说，你必须可以创建一个`class CustomPromise extends Promise{..}`子类，在这个基础上你可以扩展内置promises的功能。
例如，你需要一个自定义的promise，这个promise可以处理超过一条消息。至少理论上，实现这个只需要你继承内置`Promise`类然后扩展它。
鉴于我对JS中类概念的偏见，我认为`Promise`子类化是一种没有意义的闹剧或者转移注意力的幌子。我努力让自己想出一些Promise子类化的好处，可是我实在想不出来。
而且，如果要继续保持一些特性来遵循[Promises/A+ Test Suite][promises test],这些子类的实现很可能变得相当笨拙。
最后，我对于promise的子类化没有任何好感。

####怎么办呢！？

不涉及太多JS的细节，把`Promise`表达成一个可以被继承的"class"需要你把实例方法加入到`Promise.prototype`对象中。
但是当你这么做的时候，你就把`then..()`和`catch(..)`变成共享方法，所有`Promise`实例都可以访问到，然后这些方法只能通过this访问每个实例上的公共属性。
换句话说，如果要使得promise可以子类化，只使用简单的JS是不可能的，必须使用闭包或其他方法来为每个实例创建私有的promise状态。
我知道现在你已经开始想各种你见过的可以实现闭包私有和`this`公共继承混合的方法。
我可以写一整本书来说明为什么这样行不通，但是我这里就简单的说下：不要管你所听到的，只使用ES5中可以使用的方法，你是不可能创建私有状态同时又可以有效子类化的promise。
这两个概念在ES5以下是互相排斥的。

####Promise 削弱

另一个ES6中的新特性是**WeakMap**。简单的说，一个`WeakMap`实例能够使用对象引用作为键，然后和一个数据相联系，而不需要真正把数据存储在对象上。
这正是我们需要的，不是吗？我们需要一个我们公共的`then(..)`和`catch(..)`可以访问的`WeakMap`，无论this绑定的是什么，它们都可以根据`this`访问到并且查找对应的被保护的状态值。这个特权`Promise`方法可以取得这个内部状态，但是外部不能。
不过，事情并没有这么美好：
1. `WeakMap`根本不可能通过原生JS用性能可接受的方法实现。
2. 就算我们在ES5及以下可以使用`WeakMap`，它还是没有完全解决子类化的问题，因为你必须隐藏`WeakMap`实例使得只有你的`Promise`方法可以访问，但是这样的话另一个`Promise`的子类也能访问到。
假设我们可以解决第二个问题---其实我们不能，就做一个假设。那么`WeakMap`的实现应该是什么样的呢？
```js
var WeakMap = function(){
    var objs = [], data = [];

    function findObj(obj) {
        for (var i=0; i<objs.length; i++) {
            if (objs[i] === obj) return i;
        }

        // not found, add it onto the end
        objs.push( obj );
        data.push( undefined );

        return i;
    }

    function __set(key,value) {
        var idx = findObj( key );
        data[idx] = value;
    }

    function __get(key) {
        var idx = findObj( key );
        return data[idx];
    }

    return {
        "set": __set,
        "get": __get
    };
};

var myMap = new WeakMap();
var myObj = {};

myMap.set( myObj, "foo" );

myObj.foo; // undefined

myMap.get( myObj ); // "foo"
```
OK，基本的思想就是我们维护两个数组(objs，data)，通过下标相对应。在第一个数组中保存对象引用，在另一个保存数据。
漂亮，不是吗？
看看性能怎么样吧。看看`findObj(..)`，它要循环整个数组来找到相应的数据。引用越多性能就越低。
但是这还不是最坏的地方。`WeakMap`之所以叫做“Weak”是由于垃圾回收行为。在我们`WeakMap`的实现中，会保存每个对象的引用，这就意味着就算程序已经没有对于对象的引用了，这些对象还是不能被回收。但是真正的`WeakMap`就是这么“weak”，所以你不需要做任何事情来优化垃圾回收。
好的，`WeakMap`是一个错误的希望。它并没有解决ES6中的问题并且使得事情在ES5及以下变得更糟。

####保护state还是子类化？

这是个问题！
我真的希望我能创建一个忠实的`Peomise polyfill`给ES5及以下。
但是必须做一个选择，在这里出现了一个分歧。要不就放弃子类化的功能，要不就放弃作为promise的可靠性。
那么我们该怎么做呢？

总结
---
我会做另一个promise polyfill，这个polyfill选择保留子类化的能力，以可变的state为代价。
我已经选择了抛弃子类化使得我的promise polyfill可以很可靠。
就像我之前说的，我认为promise的子类化最终会被证明是一个华而不实的东西。我不会牺牲promise的可靠性来顺从子类化。
很显然，其他人对于这个问题会有不同的看法。但是我只想让你问问你自己：一个不可靠的promise可以用来干嘛？什么代码能真正拯救你？什么代码可以做得更好？
现有的Promise polyfill和库的问题比不可变的state vs 子类化更深层面。在第四部分：扩展问题中，我会指出许多现有polyfill和库中的问题。

译者注
---
这篇文章不大好翻译也不大好理解，所以在这里总结下我的理解，希望对大家的理解有所帮助，如果大家有什么不同的看法，欢迎讨论。
这篇文章围绕Promise的可靠性展开，Promise的可靠性是它的精髓所在。要实现Promise的可靠性最关键的就是要保证Promise的状态值state不能被外部改变，这样才能保证状态值的不可逆。
而现在几乎所有的Promise库都忽略了这个关键，而它们会忽略这个关键点一个很重要的原因就是在ES6的规范中，Promise被规定为一个类，也就是说Promise是可以被子类化的。然而在ES5及以下的规范中，在没有`private`关键字的情况下，是不可能实现可子类化同时又能保证Promise的状态值不会被外部改变（真的吗？我保持怀疑态度）。而在ES6中出现的新对象`WeakMap`确实给实现Promise带来了新的思路，可以在ES5及以下环境中实现`WeakMap`，利用它的特点可以实现符合要求的Promise。具体实现思路就是：定义一个全局私有的`WeakMap`，这个`WeakMap`只有公共的方法`then()`和`catch()`可以访问到，在这个`WeakMap`中以每个Promise实例的this作为键，状态值state作为值进行存储。这样在每个Promise实例中都可以通过自己的this对象查找自己的状态值，而不能查找到其他Promise实例的状态值，这样就实现了状态值的外部不可修改。但是`WeakMap`有一个很大的问题就是性能比较低并且不利于垃圾回收，所以这并不是一个理想的解决方案。
综上两个原因就导致了现在大部分库暴露state状态值，它们为了实现子类化选择了暴露状态值，丢弃了Promise的精髓所在。
而在作者看来子类化对于Promise的重要性远远比不上Promise的可靠性，所以它选择了放弃子类化而保证Promise的可靠性。事实确实是这样，如果不能保证Promise的可靠性，那么就会出现[第一篇](https://github.com/WHUWuShuang/work_doc/blob/master/learn/js/Promises_The%20Trust%20Problem(part1).md)中出现的那个不可靠的情况，这样Promise除了改善了回调金字塔的问题，跟普通的回调也就没有什么区别了，也就失去了它更重要的意义。

[深入理解Promise五部曲--1.异步问题](http://blog.getify.com/promises-part-1/)  
[深入理解Promise五部曲--2.转换问题](http://blog.getify.com/promises-part-2/)  
[深入理解Promise五部曲--3.可靠性问题](http://blog.getify.com/promises-part-3/)  
[深入理解Promise五部曲--4.扩展性问题](http://blog.getify.com/promises-part-4/)  
[深入理解Promise五部曲--5.乐高问题](http://blog.getify.com/promises-part-5/)

原文地址：http://blog.getify.com/promises-part-3/

[promises test]: https://github.com/promises-aplus/promises-tests