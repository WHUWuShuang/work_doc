在[Part4:扩展问题中]((https://github.com/WHUWuShuang/work_doc/blob/master/learn/js/Promises_The%20Trust%20Problem(part4).md)，我讨论了如何扩展和抽象Promise是多么的常见，以及这中间的一些问题。但是为什么promise对于开发者来说不是足够友好的呢？这就是它的设计用意吗？

>##I've Got Friends In Low Places

Promise被设计为低级别的构建块。一个promise就像一个乐高玩具。单个乐高只是一个有趣的玩具。但是如果把它们拼在一起，你会感受到更多的乐趣。
问题是promise不是你小时候玩儿的那个有趣LEGO，它们不是充满想象力的打气筒，也不是Gandalf mini-figure(一种乐高玩具)。
都不是，promise只是你的简单老旧的4X2的砖块。
这并不是使它们非常有用。但是它们是你箱子中最重要的组成部分之一。当它们彼此分开时它们只是这么个东西，但是当把它们整合在一起它们就会散发出光芒。
换句话说，promise本质上是一个构建在真实用户之上的低级别的API。这是对的：__promise并不是给开发者使用的__，它们是给库作者使用的。
你会从它们那收益许多，但是你很可能不是直接使用它们。你将会使用的是经过许多库组合包装之后的结果。

>##控制 VS 值

请允许我矫正第一个最大的关于promise的误解：__它们不是真正关于流程控制的__。
promise当然可以链接在一起来变成近似异步流程控制的东西。但是最后证明它们并不像你想象的那样擅长这个任务。
promises确实只是一个值的容器。这个值可能现在就存在也可能是未来的一个值。但是不管怎样，它只是一个值。
这是promise最有意义的好处之一。它们在值的上面创建了一个强大的抽象使得值不再是暂存的东西。换句话说，不管那个值现在是否存在，你都可以用同样的方式使用promise。在这个系列的[第三部分]((https://github.com/WHUWuShuang/work_doc/blob/master/learn/js/Promises_The%20Trust%20Problem(part3).md)中，我讨论过promise必须是不可变的，它们作为值的意义也是基于这个特点的。
promises就像状态的小型的自包含的表现方式。它们是可组合的，也就意味着你全部的程序可以用它们来表示。

>##限制

就像你不能奢望一个单独的4X2的乐高可以变成一个跑车，让promise成为你的异步流程控制机制也是一种奢望。
那么promises作为一个非暂存的不可变的值对于解决异步任务意味着什么呢？在它们设计哲学的约束中，有它们擅长并且可以有帮助的东西。
在剩下的内容中，我会讨论这个限制。但是我并不打算作为一个promise的批判者。我试图去强调扩展和抽象的重要性。

错误处理
---
当我说promise只是一个值的容器的时候我撒了个小慌。实际上，它是一个成功值或者失败信息的容器。在任何时候，一个promise是一个未来的成功值或者在获取这个值时的失败信息。不会超过这两种情况。
在某种意义上说，一个promise是一个决策结构，一个`if..then..else`。其他人喜欢把它想成一个`try..catch`结构。不管是哪种理解，你就像在说"请求一个值，不管成功还是失败"。
就像尤达说，"Do or do not, there is no try."。
考虑下面这个情况：
```js
function ajax(url) {
    return new Promise( function(resolve,reject){
        // make some ajax request
        // if you get a response, `resolve( answer )`
        // if it fails, `reject( excuses )`
    } );
}

ajax( "http://TheMeaningOfLife.com" )
.then(
    winAtLife,
    keepSearching
);
```
看到`winAtLife()`和`keepSearching()`函数了吗？我们在说，"去问问生命的意义，不管你有没有找到答案，我们都继续"。
如果我们不传入`keepSearching`会怎样？除了作为一个乐观主义者假设你会找到答案然后在生命长河中取胜，这里会有什么危险呢？
如果promise没有找到生命的意义（或者如果在处理答案的过程中发生了javascript异常），它会默默地保留着错误的事实，也许会永远保留着。就算你等上一百万年，你都不会知道对于答案的请求失败了。
你只能通过观察才能知道它失败了。这可能需要深入到形而上学或者量子学的东西。让我们停止在这吧。
所以不带失败处理函数的promise是一个会默默地失败的promise。这并不好。这意味着如果你忘记了，你会陷入失败的陷阱而不是成功。
所以你会怀疑：为什么promises会忽略失败处理函数呢？因为你可能现在不在意失败的情况，只有以后某个时刻会关心。我们程序的暂时性意味着系统现在不会知道你以后会想做什么。现在忽略失败处理函数也许对你来说是正合适的，因为你知道你会把这个promise链接到另一个promise，并且那个promise有一个失败处理函数。
所以promise机制让你可以创建不需要监听失败的promise。
这里有一个很微妙的问题，很可能也是大多数刚接触promise的开发者会碰到的问题。

束缚我们的链子
---
为了理解这个问题，我们首先需要理解promises是如何链接在一起的。我认为你会很快明白promise链是强大并且有一点复杂的。
```js
ajax( "http://TheMeaningOfLife.com" )
.then(
    winAtLife,
    keepSearching
)
// a second promise returned here that we ignored!
;
```
`ajax(..)`调用产生了第一个promise，然后then(..)调用产生了第二个promise。我们没有捕捉并且观察在这段代码中的第二个promise，但是我们可以。第二个promise是根据第一个promise处理函数如何运行来自动变成fulfilled状态(成功或者失败)。
第二个promise不会在意第一个promise是成功还是失败。它在意第一个promise的处理函数(不管成功还是失败)。
这是promise链的关键。但是这有一点不好理解，所以重复读上面那段话直到你理解为止。
考虑下promise代码通常是怎么写的(通过链):
```js
ajax( ".." )
.then( transformResult )
.then(
    displayAnswer,
    reportError
);
```
这段代码也可以像下面这么写，效果是一样的：
```js
var promiseA = ajax( ".." );

var promiseB = promiseA.then( transformResult );

var promiseC = promiseB.then(
    displayAnswer,
    reportError
);

// we don't use `promiseC` here, but we could...
```
Promise A是唯一在意`ajax(..)`结果的promise。
Promise B只关心Promise A在`transformResult(..)`函数内部是如何处理的(不是Promise A的结果本身),同样的，Promise C只关心Promise B在`displayAnswer(..)`或者`reportError(..)`函数内部是如何处理的(不是Promise B结果本身)。
再一次，重复读这段话直到理解。
在`transformResult(..)`内部，如果它立刻完成了它的任务，然后Promise B就会立刻完成，不管成功还是失败。然而，如果`transformResult(..)`不能立刻完成，而是创建它自己的promise，我们称它为Promise H1('H'是'hidden',因为它是隐藏在内部的)。原本Promise B返回的等待我们如何处理Promise A的promise，现在概念上被Promise H1替换了(并不是真的替换了，只是被说成一样的)。
所以，现在当你说`promiseB.then(..)`时，它实际上就像说`promiseH1.then(..)`。如果Promise H1成功了，`displayAnswer(..)`会被调用，但是如果它失败了，`reportError(..)`会被调用。
这就是promise链是如何工作的。
但是，如果Promise A(由ajax调用返回)失败了会怎样？`promiseA.then(..)`调用没有注册一个失败处理函数。它会默默地隐藏错误吗？它会的，除了我们链接上Promise B然后在上面注册一个错误处理函数:`reportError(..)`。如果Promise A失败了，`transformResult(..)`不会被调用，并且没有错误处理函数，所以Promise B马上被标记为失败，所以`reportError(..)`会被调用。
如果Promise A成功了，`transformResult(..)`会被执行，然后当运行`transformResult(..)`时有一个错误会怎样？Promise B被标记为失败，然后`reportError(..)`也会被调用。
但是这里是危险的地方，这个地方甚至有经验的开发者都会遗漏的！
如果Promise A成功了(成功的`ajax(..)`)，然后Promise B成功了(成功的`transformResult(..)`)，但是当运行`displayAnswer(..)`时有一个错误会怎样？
你也许会认为`reportError(..)`会被调用？大多数人会这么想，但是不是的。
为什么？因为来自`displayAnswer(..)`的一个错误或者失败promise导致一个失败的Promise C。我们监听Promise C失败的情况了吗？仔细看看。没有。
为了确保你不会漏掉这种错误并且让它默默地隐藏在Promise C状态内部，你也会希望监听Promise C的失败：
```js
var promiseC = promiseB.then(
    displayAnswer,
    reportError
);

// need to do this:
promiseC.then( null, reportError );

// or this:, which is the same thing:
promiseC.catch( reportError );

// Note: a silently ignored *Promise D* was created here!
```
OK,所以现在我们捕获`displayAnswer(..)`内部的错误。不得不去记住这个有一点坑爹。

乌龟
---
但是有一个更加微妙的问题！如果当处理`displayAnswer(..)`返回的错误时，`reportError(..)`函数也有一个JS异常会怎样？会有人捕获这个错误吗？没有。
看！上面有一个隐含的Promise D，并且它会被告知reportError(..)内部的异常。
OMG，你肯定会想。什么时候才能停止？它会这样一直下去吗？
一些promise库作者认为有必要解决这个问题通过让"安静的错误"被作为全局异常抛出。但是这种机制该如何得知你不想再链接promise并且提供一个错误处理函数呢？它如何知道什么时候应该通报一个全局异常或者不通报呢？你肯定不希望当你已经捕获并且处理错误的情况下仍然有很多控制台错误信息。
在某种意义上，你需要可以标记一个promise为“final”，就像说“这是我链子中的最后一个promise”或者“我不打算再链接了，所以这是乌龟停止的地方”。如果在链的最后发生了错误并且没有被捕获，然后它需要被报告为一个全局异常。
从表面上我猜测这似乎是很明智的。这种情况下的实现像下面这样：
```js
var promiseC = promiseB.then(
    displayAnswer,
    reportError
);

promiseC
.catch( reportError )
.done(); // marking the end of the chain
``
你仍然需要记住调用`done()`，要不然错误还是会隐藏在最后一个promsie中。你必须使用稳固的错误处理函数。
"恶心"，你肯定会这么想。欢迎来到promises的欢乐世界。

Value vs Values
---
对于错误处理已经说了很多了。另一个核心promsie的限制是一个promise代表一个单独的值。什么是一个单独的值呢？它是一个对象或者一个数组或者一个字符串或者一个数字。等等，我还可以在一个容器里放入多个值，就像一个数组或对象中的多个元素。Cool！
一个操作的最终结果不总是一个值，但是promise并不会这样，这很微妙并且又是另一个失败陷阱：
```js
function ajax(url) {
    return new Promise( function(resolve,reject){
        // make some ajax request
        // if you get a response, `resolve( answer, url )`
        // if it fails, `reject( excuses, url )`
    } );
}

ajax( ".." )
.then(
    function(answer,url){
        console.log( answer, url ); // ..  undefined
    },
    function(excuses,url){
        console.log( excuses, url ); // ..  undefined
    }
);
```
你看出这里面的问题了吗？如果你意外的尝试传递超过一个的值过去，不管传给失败处理函数还是成功处理函数，只有第一个值能被传递过去，其他几个会被默默地丢掉。
为什么？我相信这和组合的可预测性有关，或者一些其他花哨的词汇有关。
最后，你不得不记住包裹自己的多个值要不然你就会不知不觉的丢失数据。

并行
---
真实世界中的app经常在“同一时间”发生超过一件事情。本质上说，我们需要构建一个处理器，并行处理多个事件，等待它们全部完成再执行回调函数。
相比于promise问题，这是一个异步流程控制的问题。一个单独的promise不能表达两个或更多并行发生的异步事件。你需要一个抽象层来处理它。
在计算机科学术语中，这个概念叫做一个“门”。一个等待所有任务完成，并且不关心它们完成顺序的门。
在promise世界中，我们添加一个API叫做`Promise.all(..)`，它可以构建一个promise来等待所有传递进来的promise完成。
```js
Promise.all([
    // these will all proceed "in parallel"
    makePromise1(),
    makePromise2(),
    makePromise3()
])
.then( .. );
```
一个相近的方法是`race()`。它的作用和`all()`一样，除了它只要有一个promise返回消息就执行回调函数，而不等待其他promise的结果。
当你思考这些方法的时候，你可能会想到许多方式来实现这些方法。`Promise.all(..)`和`Promise.race(..)`是原生提供的，因为这两个方法是很常用到的，但是如果你还需要其他的功能那么你就需要一个库来帮助你了。
限制的另一个表现就是你很快就会发现你需要自己使用`Array`的相关方法来管理promise列表，比如`.map(..)`和`.reduce(..)`。如果你对map/reduce不熟悉，那么赶紧去熟悉一下，因为你会发现当处理现实世界中promise的时候你经常会需要它们。
幸运的是，已经有很多库来帮助你了，并且每天还有很多新的库被创造出来。

Single Shot Of Espresso，Please！
--
另一个关于promise的事情是它们只会运行一次，然后就不用了。
如果你只需要处理单个事件，比如初始化一个也没或者资源加载，那么这样没什么问题。但是如果你有一个重复的事件(比如用户点击按钮)，你每次都需要执行一系列异步操作会怎么样呢？
Promise并不提供这样的功能，因为它们是不可变的，也就是不能被重置。要重复同样的promise，唯一的方法就是重新定义一个promise。
```js
$("#my_button").click(function(evt){
    doTask1( evt.target )
    .then( doTask2 )
    .then( doTask3 )
    .catch( handleError );
});
```
太恶心了，不仅仅是因为重复创建promise对于效率有影响，而且它对于职责分散不利。你不得不把多个事件监听函数放在同一个函数中。如果有一个方式来改变这种情况就好了，这样事件监听和事件处理函数就能够分开了。
Microsoft的RxJS库把这种方式叫做"观察者模式"。
我的_asynquence_库有一个`react(..)`方法通过简单的方式提供了一个类似的功能。

盲区...
---
在一个已经被使用回调函数的API占据的世界中，把promise插入到代码中比我们想象的要困难。考虑下面这段代码：
```js
function myAjax(url) {
    return new Promise( function(resolve,reject){
        ajax( url, function(err,response){
            if (err) {
                reject( err );
            }
            else {
                resolve( response );
            }
        } )
    } );
}
```
我认为promise解决了回调地狱的问题，但是它们代码看起来仍然像垃圾。
我们需要抽象层来使得用promise表示回调变得更简单。原生的promise并没有提供这个抽象层，所以结果就是通过原生promise写出来的代码还是很丑陋。但是如果有抽象层那么事情就变得很简单了。
例如，我的_asynquence_库提供了一个[errfcb()插件](https://github.com/getify/asynquence/tree/master/contrib#errfcb-plugin)(error-first callback)，用它可以构建一个回调来处理下面这种场景:
```js
function myAjax(url) {
    var sq = ASQ();
    ajax( url, sq.errfcb() );
    return sq;
}
```
Stop The Presses!
---
有时，你想要取消一个promise而去做别的事情，但是如果现在你的promise正处在挂起状态会怎样呢？
```js
var pr = ajax( ".." )
.then( transformResult )
.then(
    displayAnswer,
    reportError
);

// Later
pr.cancel(); //  <-- doesn't work!
```
所以，为了取消promise，你需要引入一下东西：
```js
function transformResult(data) {
    if (!pr.ignored) {
        // do something!
    }
}

var pr = ajax( ".." )
.then( transformResult )
.then(
    displayAnswer,
    reportError
);

// Later
pr.ignored = true; // just hacking around
```
换句话说，你为了能够取消你的promise，在promise上面加了一层来处理这种情况。
你不能从promise取消注册处理函数。并且因为一个promise必须不可变，你能够直接取消一个promise这种情况是不允许出现的。从外部取消一个promise跟改变它的状态没有什么区别。它使得promise变得不可靠。
许多promise库都提供了这种功能，但是这明显是一个错误。取消这种行为是不需要promise，但是它可以出现在promise上面的一个抽象层里。

冗长
---
另一个关于原生promise的担心是有些事情并没有被实现，所以你必须自动手动实现它们，而这些事情对于可扩展性是很重要的，但是这些东西经常会导致令人讨厌的重复代码。
看一个例子，在每一个promise的完成步骤中，有一个设定就是你希望保持链式结构，所以`then(..)`方法会返回一个新的promise。但是如果你想要加入一个自己创建的promise并且从一个成功处理函数中返回，这样你的promise就可以加入到链的流程控制中。
```js
function transformResult(data) {
    // we have to manually create and return a promise here
    return new Promise( function(resolve,reject){
        // whatever
    } );
}

var pr = ajax( ".." )
.then( transformResult )
.then(
    displayAnswer,
    reportError
);
```
不同的是，就像上面解释的一样，从第一个`then(..)`返回的隐藏的promise立刻就完成(或者失败)，然后你就没办法让剩下的链异步延迟。
如果有一个抽象层能够通过某种方式把自动创建/链接的promise暴露给你，然后你就不需要创建自己的promise来替换了，这样该多好。
换句话说，如果有一个设定假设你需要为了异步的目的使用链，而不是你只是需要漂亮得执行异步。（也就是说你确实是希望你的代码可以异步执行，而不是说希望整个异步流程看过去好看点）。
另一个例子：你不能直接传递一个已经存在的promise给`then(..)`方法，你必须传递一个返回这个promise的函数。
```js
var pr = doTask2();

doTask1()
.then( pr ); // would be nice, but doesn't work!

// instead:

doTask1()
.then( function(){ return pr; } );
```
这个限制性是有很多原因的。但是它只是减弱了有利于保持可扩展性和可预测性的用法的简洁。抽象可以容易的解决这个问题。

>##全剧终

所有这些原因就是为什么原生的promise API是强大同时也是有局限性的。
关于扩展和抽象是一个成熟的领域。许多库正在做这些工作。就像我之前说的，[asynquence](http://github.com/getify/asynquence)是我自己的promise抽象库。它很小但是很强大。它解决了所有博客中提到的promise的问题。
我后面会写一篇详细的博客来介绍asynquence是如果解决这些问题的，所以敬请期待。

[深入理解Promise五部曲--1.异步问题](http://blog.getify.com/promises-part-1/)  
[深入理解Promise五部曲--2.转换问题](http://blog.getify.com/promises-part-2/)  
[深入理解Promise五部曲--3.可靠性问题](http://blog.getify.com/promises-part-3/)  
[深入理解Promise五部曲--4.扩展性问题](http://blog.getify.com/promises-part-4/)  
[深入理解Promise五部曲--5.乐高问题](http://blog.getify.com/promises-part-5/)

原文地址：http://blog.getify.com/promises-part-5/