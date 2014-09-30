现在，我希望你已经看过深入理解Promise的前三篇文章了。并且假设你已经完全理解Promises是什么以及深入讨论Promises的重要性。

>##不要扩展原生对象！

回到2005年，[Prototype.js](http://prototypejs.org/)框架是最先提出扩展Javascript原生对象的内置prototype属性的框架之一。它们的想法是我们可以通过向prototype属性添加额外的方法来扩展现有的功能。
如果你对近十年Javascript编程做一个简单的调查，比如使用google简单搜索下，你会发现对于这个想法有很多反对意见。它们都是有原因的。
大多数开发者会告诉你：“不要扩展原生对象”或者“只在polyfill的时候扩展原生对象”。后者意味着只有当扩展的功能已经被列入规范然后你只是为了能在旧的环境中使用这些功能的时候才能对元素对象进行扩展。

数组Push方法
---
想象一个真实的场景(确实发生在我身上)：回到Netscape3/4和IE4的时代，当时的JS并没有现在这么友好。作为许多显著差异中的一个，数组并没有`push(..)`方法来向它的尾部添加元素。
所以，一些人会通过下面这段代码来扩展：
```js
//Netscape 4 doesn't hava Array.push
Array.prototype.push = function(elem){
    this[this.length - 1] = elem ;
}
```
乍一看可能觉得没问题，但是你很快就会发现一些问题。

1.  这里需要一个if来判断原生是否有对于`push(..)`的支持，如果有我们就可以使用原生的`push(..)`方法。
2.  我们需要注意我们已经破坏了数组对象`for..in`循环行为，因为我们的`push(..)`方法会出现在`for..in`循环的结果中。当然，你不应该在数组上使用`for..in`循环，这又是另外一个话题了。

有一个和1相关的更大的问题。不仅仅是需要一个`if`判断：
```js
if(!Array.prototype.push){
    //make our own
}
```
我们应该问问我们自己，如果内置的`push(..)`实现和我们的实现不兼容怎么办？如果内置的实现接受不一样数量的参数或者不一样的参数类型怎么办？
如果我们的代码依赖于我们自己实现的`push(..)`，然后我们只是简单的用新的方法替换我们自己的方法，那么代码会出现问题。如果我们的实现覆盖了内置的`push(..)`实现，然后如果一些JS库期望使用内置的标准`push(..)`方法怎么办？
这些问题是真实发生在我身上的。我有一个工作是在一个用户的古老的网站上加入一个组件，然后这个组件依赖于jQuery。我们组件在其他网站都可以正常使用，但是在这个特殊的站点却无法使用。我花了很多时间来找出问题。
最终，我定位到了上面那个`if`代码片段。这里有什么问题呢？
它的`push(..)`实现只接受一个参数，然而jQuery中期望是通过push(el1,el2,...)来调用push方法，所以它就无法正常运行了。
Oops。
但是猜猜当我移除原来的push代码时发生了什么？在其他网站这个组件也不能使用的。为什么？我还是不知道具体是为什么。我认为他们意外地依赖于外部变量，而这些外部变量没有传递进来。
但是，真正的问题是，有人通过一种对于未来存在潜在危险的方式扩展内置原生对象，导致这个方法在未来可能无法正常运行。
我不是唯一遇到这个问题的人。成千上万的开发者都遇到了这种情况。我们中的大多数认为你必须十分小心当你扩展原生JS对象时。如果你这么做了，你最好不要使用跟语言新版本中的方法名相同的名字。

>##Promise扩展

为什么所有的老爷爷抱怨如今Promises的火热呢？
因为那些开发`Promise`"polyfills"的人似乎忘记或者抛弃了老人们的智慧。他们开始直接往`Promise`和`Promsie.prototype`上加额外的东西。
我真的需要再去解释为什么这是一个“未来的”坏点子吗？

Blue In The Face
----------
我们可以一直争论这个问题到死，但是仍然不能改变这个事实。如果你扩展原生对象，你就是和未来敌对的，就算你觉得你自己已经做得很好了。
而且，你用越大众化的名字来扩展原生对象，你越有可能影响未来的人。
让我们看看[Bluebird](https://github.com/petkaantonov/bluebird)库，因为它是最流行的Promise polyfill库之一。它足够快但是它跟其他库比起来也更加大。
但是速度和大小并不是我现在担心的。我关心的是它选择了把自己添加到`Promise`的命名空间上。就算它使用一个polyfill安全的方式，实际上并没有，事实就算它添加许多额外的东西到原生对象上。
例如，Bluebird添加了`Promise.method(..)`:
```js
function someAsyncNonPromiseFunc() {
    // ...
}

var wrappedFn = Promise.method( someAsyncNonPromiseFunc );

var p = wrappedFn(..);

p.then(..)..;
```
看起来没什么问题，是吗？当然。但是如果某天规范需要添加`Promise.method(..)`方法。然后如果它的行为和Bluebird有很大的区别会怎么样呢？
你又会看到`Array.prototype.push(..)`一样的情况。
Bluebird添加了许多东西到原生的`Promise`。所以有很多可能性会在未来会发生冲突。
我希望我从来不需要去修复某个人的`Promise`扩展代码。但是，我很可能需要这么做。

未来约束
--------------------
但是这还不是最糟的。如果Bluebir非常流行，然后许多现实中的网站依赖于这么一个扩展，突然一天TC39协会通过某种方式强制避免扩展官方规范，那么这些依赖于扩展的网站都将崩溃。
你看，这就是扩展原生对象的危险所在：你为了实现你的功能然后扩展原生对象，然后就拍拍屁股把这些烂摊子留给了TC39成员们。因为你愚蠢的决定Javascript的维护者只能选择其他机制。
不相信我？这种情况已经发生很多次了。你知道为什么在19年的JS历史中`typeof null === "object"`这个bug一直无法修复吗？因为太多的代码都依赖于这段代码了。如果他们修复了这个bug，那么结果可想而知。
我真的不想这种事情发生在Promise身上。请停止通过扩展原生对象来定义Promise polyfill库。

包装抽象
---
我认为我们需要更多不破坏规范的polyfill，像我的"[Native Promise Only](https://github.com/getify/native-promise-only)"。我们需要良好，稳固，性能优秀但是和标准兼容的polyfill。
特别的，我们需要它们以便于那些需要扩展promise的人可以在这个包装上进行操作。我们不应该很容易获得一个`Promise polyfill`然后创建我们自己的`SuperAwesomePromise`包装在它上面吗？
已经有很多的好例子了，比如[Q](https://github.com/kriskowal/q)和[when](https://github.com/cujojs/when)
我自己也写了一个，叫做[asnquence](https://github.com/getify/asynquence)(async + sequence)
我的是设计来隐藏promises的，因为promise是低级别的API，所以与其给你一个简单的抽象的东西不如隐藏丑陋的细节。
例如，比较下下面两段代码
原生Promises：
```js
function delay(n) {
    return new Promise( function(resolve,reject){
        setTimeout( resolve, n );
    } );
}

function request(url) {
    return new Promise( function(resolve,reject){
        ajax( url, function(err,res){
            if (err) reject( err );
            else resolve( res );
        } );
    } );
}

delay( 100 )
.then( function(){
    return request( "some/url" );
} )
.then(
    success,
    error
);
```
asynquence：
```js
function delay(n) {
    return ASQ( function(done){
        setTimeout( done, n );
    } );
}

function request(url) {
    return ASQ( function(done){
        ajax( url, done.errfcb );
    } );
}

delay( 100 )
.val( "some/url" )
.seq( request )
.then( success )
.or( error );
```
希望你能够通过这个简单的例子看出_asynquence_是如何降低使用promises来表达异步流程的难度的。它在底层实现为你创建promise，它自动把它们连接在一起，然后为同样的组合模式提供了简单的语法。
显然，我认为_asynquence_是非常令人惊奇的。我认为你应该看看一些[例子](https://github.com/getify/asynquence/blob/master/contrib/README.md)，然后看看大家扩展的[插件](https://github.com/getify/asynquence/blob/master/contrib/README.md)，这些插件使得它能提供更多的便利。
如果_asynquence_不是你的菜，那么你可以再寻找一个适合你的好用知名的抽象库。
但是请不要使用那些扩展原生`Promise`的库。这对于未来不是一件好事。

>##总结

Promise是令人惊奇的并且它们正在改变许多JS开发者编写和维护一部流程的方式。
ES6带来的原生Promise是这个语言一个重大的胜利。为了加速这个胜利的过程，我们中的许多人开发Promise polyfill和Promise库。
但是不要因为Promise带来的兴奋和喜悦让你忘了一个不可否认的事实：__扩展原生对象是一件危险并且充满冒险的事情__，并仅仅对于库的作者也包括使用这些库的所有人。
最后，请负有责任感并且使用安全的promise扩展。我们在将来会感谢你的。

[深入理解Promise五部曲--1.异步问题](http://blog.getify.com/promises-part-1/)  
[深入理解Promise五部曲--2.转换问题](http://blog.getify.com/promises-part-2/)  
[深入理解Promise五部曲--3.可靠性问题](http://blog.getify.com/promises-part-3/)  
[深入理解Promise五部曲--4.扩展性问题](http://blog.getify.com/promises-part-4/)  
[深入理解Promise五部曲--5.乐高问题](http://blog.getify.com/promises-part-5/)

原文地址：http://blog.getify.com/promises-part-4/