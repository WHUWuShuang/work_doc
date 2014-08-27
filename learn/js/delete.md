>#####在javascript中，我们有时候要使用delete删除对象。但是，对于delete的一些细节我们未必尽知。昨天，看到kangax分析delete的文章，获益匪浅。本文将文章的精华部分翻译出来，与各位分享。  

       
###**目录**  
*  [原理](.#原理)   
1.	[代码类型](.#代码类型)    
1.	[执行上下文](.#执行上下文)
1.	[激活对象/可变对象](.#激活对象/可变对象)
1.	[属性特性](.#属性特性)
1.	[内置对象和DontDelete](.#内置对象和DontDelete)
1.	[未声明的赋值](.#未声明的赋值)
1.	[Firebug 困惑](.#Firebug&amp;困惑)
1.	[通过eval删除变量](.#通过eval删除变量)
1.	[浏览器兼容性](.#浏览器兼容性)
1.	[Gecko DontDelete bug](.#Gecko&amp;DontDelete&amp;bug)
1.	[IE bugs](.#IE&amp;bugs)
*  [误区](.#误区)
1.	['delete’和宿主对象](.#'delete’和宿主对象)
1.	[ES5严格模式](.#ES5严格模式)
*  [总结](.#总结)

###**原理**

>为什么我们能删除一个对象的属性？
`var o = { x: 1 }; 
delete o.x; // true 
o.x; // undefined`
但是，像这样声明的变量则不行：
>`var x = 1; 
delete x; // false 
x; // 1 `
或者如此声明的函数：
>`function x(){} 
delete x; // false 
typeof x; // "function" `
注意，当一个属性不能被删除时，delete只返回false。

>要理解这一点，我们首先需要掌握像变量实例化和属性特性这样的概念－－遗憾的是这些在关于javascript的书中很少讲到。我将在接下来的几个段落中试着简明的重温这些概念。　理解它们一点也不难，如果你不在乎它们为什么这么运行，你可以随意的跳过这一章。

>####**代码类型**  

>>在ECMAScript中有三种类型的可执行代码：全局代码*（Global code）*、函数代码*（Function code）*和Eval code。这些类型有那么点自我描述，但这里还是作一个简短的概述：

>>当一段源代码正文被视为程序时，它在全局作用域中执行，被当成全局代码*（Global code）*。在一个浏览器环境中，SCRIPT元素中的内容通常被当作程序来解析，因此，它被当作全局代码来评估。   
在一个函数内部直接执行的任何代码，很明显被当作函数代码*（Function code）*。在浏览器红中事件属性的内容（如：`<p onclick="...">`）通常被当作函数代码*（Function code）*来解析。  
最后，提供给内置函数eval()的文本被当作Eval 代码（Eval code）来解析。我们很快会看到这种类型很特殊。
 
>####**执行上下文**

>>当ECMAScript代码执行时，它总是在一定的上下文中运行，执行上下文是一个有点抽象的实体，它有助于我们理解作用域和变量实例化如何工作的。对于三种类型的可执行代码，每个都有执行的上下文。当一个函数执行时，可以说控制进入到函数代码*（Function code）*的执行上下文。全局代码执行时，进入到全局代码*（Global code）*的执行上下文。

>>正如你所见，执行上下文逻辑上来自一个栈。首先可能是有自己作用域的全局代码，代码中可能调用一个函数，它有自己的作用域，函数可以调用另外一个函数，等等。即使函数递归地调用它自身，每一次调用都进入一个新的执行上下文。

>####**激活对象/可变对象**  

>>每一个执行上下文在其内部都有一个所谓的可变对象。与执行上下文类似，可变对象是一个抽象的实体，一个描述变量示例化的机制。现在，最有趣的是在源代码中声明的变量和函数被当作这个可变对象的属性被添加。

>>当控制进入全局代码的执行上下文时，一个全局对象用作可变对象。这也正是为什么在全局范围中声明的变量或者函数变成了全局对象的属性。

>>`/* remember that 'this' refers to global object when in global scope */
var GLOBAL_OBJECT = this;
var foo = 1;
GLOBAL_OBJECT.foo; // 1
foo === GLOBAL_OBJECT.foo; // true
function bar(){}
typeof GLOBAL_OBJECT.bar; // "function"
GLOBAL_OBJECT.bar === bar; // true`
Ok，全局变量变成了全局对象的属性，但是，那些在函数代码*（Function code）*中定义的局部变量又会如何呢？行为其实很相似：它成了可变对象的属性。唯一的差别在于在函数代码*（Function code）*中，可变对象不是全局对象，而是所谓的激活对象。每次函数代码*（Function code）*进入执行作用域时，激活对象即被创建。

>>不仅函数代码*（Function code）*中的变量和函数成为激活对象的属性，而且函数的每一个参数（与形参相对应的名称）和一个特定Arguments 对象*（Arguments ）*也是。注意，激活对象是一种内部机制，不会被程序代码真正访问到。

>>`(function(foo){
  var bar = 2;
  function baz(){}
  /*
  In abstract terms,
  Special 'arguments' object becomes a property of containing function's Activation object: 
    ACTIVATION_OBJECT.arguments; // Arguments object
  ...as well as argument 'foo':
    ACTIVATION_OBJECT.foo; // 1
  ...as well as variable 'bar':
    ACTIVATION_OBJECT.bar; // 2
  ...as well as function declared locally:
    typeof ACTIVATION_OBJECT.baz; // "function"
  */
})(1);`
最后，在Eval 代码*（Eval code）*中声明的变量作为正在调用的上下文的可变对象的属性被创建。Eval 代码*（Eval code）*只使用它正在被调用的哪个执行上下文的可变对象。

>>`var GLOBAL_OBJECT = this; 
/* 'foo; is created as a property of calling context Variable object, 
    which in this case is a Global object */
eval('var foo = 1;'); 
GLOBAL_OBJECT.foo; // 1 
(function(){ 
  /* 'bar' is created as a property of calling context Variable object, 
    which in this case is an Activation object of containing function */
  eval('var bar = 1;'); 
  /* 
    In abstract terms, 
    ACTIVATION_OBJECT.bar; // 1 
  */
})();`

>####**属性特性**

>>现在变量会怎样已经很清楚（它们成为属性），剩下唯一的需要理解的概念是属性特性。每个属性都有来自下列一组属性中的零个或多个特性－－ReadOnly, DontEnum, DontDelete 和Internal，你可以认为它们是一个标记，一个属性可有可无的特性。为了今天讨论的目的，我们只关心DontDelete 特性。

>>当声明的变量和函数成为一个可变对象的属性时－－要么是激活对象*（Function code）*，要么是全局对象*（Global code）*，这些创建的属性带有DontDelete 特性。但是，任何明确的（或隐含的）创建的属性不具有DontDelete 特性。这就是我们为什么一些属性能删除，一些不能。

>>`var GLOBAL_OBJECT = this; 
  /*  'foo' is a property of a Global object. 
      It is created via variable declaration and so has DontDelete attribute. 
      This is why it can not be deleted. */
  var foo = 1; 
  delete foo; // false 
  typeof foo; // "number" 
  /*  'bar' is a property of a Global object. 
      It is created via function declaration and so has DontDelete attribute. 
      This is why it can not be deleted either. */
  function bar(){} 
  delete bar; // false 
  typeof bar; // "function" 
  /*  'baz' is also a property of a Global object. 
      However, it is created via property assignment and so has no DontDelete attribute. 
      This is why it can be deleted. */
  GLOBAL_OBJECT.baz = 'blah'; 
  delete GLOBAL_OBJECT.baz; // true 
  typeof GLOBAL_OBJECT.baz; // "undefine` 
  
>####**内置对象和DontDelete**

这就是全部：属性中一个独特的特性控制着这个属性是否能被删除。注意，内置对象的一些属性也有特定的DontDelete 特性，因此，它不能被删除。特定的Arguments 变量（或者，正如我们现在了解的，激活对象的属性），任何函数实例的length属性也拥有DontDelete 特性。

代码 
1 function(){ 
2   /* can't delete `arguments`, since it has DontDelete */
3   delete arguments; // false 
4   typeof arguments; // "object" 
5   /* can't delete function's `length`; it also has DontDelete */
6   function f(){} 
7   delete f.length; // false 
8   typeof f.length; // "number" 
9 })();
与函数参数相对应的创建的属性也有DontDelete 特性，因此也不能被删除。

1 (function(foo, bar){ 
2   
3     delete foo; // false 
4     foo; // 1 
5   
6     delete bar; // false 
7     bar; // 'blah' 
8   
9   })(1, 'blah');
未声明的赋值

您可能还记得，未声明的赋值在一个全局对象上创建一个属性。除非它在全局对象之前的作用域中的某个地方可见。现在我们知道属性分配与变量声明之间的差异，后者设置了DontDelete 特性，而前者没有－－应该很清楚未声明的赋值创建了一个可删除的属性。

代码 
1 var GLOBAL_OBJECT = this; 
2 /* create global property via variable declaration; property has <STRONG>DontDelete</STRONG> */
3 var foo = 1; 
4 /* create global property via undeclared assignment; property has no <STRONG>DontDelete</STRONG> */
5 bar = 2; 
6 delete foo; // false 
7 typeof foo; // "number" 
8 delete bar; // true 
9 typeof bar; // "undefined"
请注意，该特性是在属性创建的过程中确定的（例如：none）。后来的赋值不会修改现有属性已经存在的特性，理解这一点很重要。

代码 
 1 /* `foo` is created as a property with DontDelete */
 2 function foo(){} 
 3 /* Later assignments do not modify attributes. DontDelete is still there! */
 4 foo = 1; 
 5 delete foo; // false 
 6 typeof foo; // "number" 
 7 /* But assigning to a property that doesn't exist, 
 8    creates that property with empty attributes (and so without DontDelete) */
 9 this.bar = 1; 
10 delete bar; // true 
11 typeof bar; // "undefined"
Firebug 困惑

那么，在Firebug中会发生什么呢？为什么在控制台中定义的变量可以被删除，难道与我们刚才了解到相反？很好，我先前说过，当涉及到的变量声明，Eval 代码（Eval code）有一个特殊的行为。在Eval 代码（Eval code）中声明的变量实际上没有创建DontDelete 特性。

1 eval('var foo = 1;'); 
2 foo; // 1 
3 delete foo; // true 
4 typeof foo; // "undefined"confusion
同样，在函数代码*（Function code）*调用也是如此：

1 (function(){ 
2   eval('var foo = 1;'); 
3   foo; // 1 
4   delete foo; // true 
5   typeof foo; // "undefined" 
6 })();
这是Firebug的异常行为的要点，在控制台的所有文本似乎是作为Eval 代码（Eval code）来解析和执行的，而不是作为一个全局对象或函数对象，显然，任何声明的变量没有DontDelete特性，因此可以很容易地删除，应该意识到正常全局代码和Firebug控制台之间的分歧。

通过eval删除变量

这个有趣的eval属性，连同ECMAScript 其它方面的技巧可以让我们删除不可删除的属性。在同一个执行上下文中，函数声明能覆盖同一名字的变量。

1 function x(){ } 
2 var x; 
3 typeof x; // "function"
注意函数如何获得优先权并覆盖同名变量（或者换句话说，可变对象相同的属性）。这是因为函数声明在变量声明之后实例化，并且可以覆盖它们。函数声明不仅取代了先前的属性值，而且也取代了属性特性。如果我们通过eval声明函数，该函数也应该替换自身的属性特性。既然在eval内声明的变量没有DontDelete特性，那么实例化这个新函数应该从本质上消除属性中现有的DontDelete特性，是的这个属性可以删除（当然也就改变引用新创建函数的值）。

代码

1 var x = 1; 
2   /* Can't delete, `x` has DontDelete */
3   delete x; // false 
4   typeof x; // "number" 
5   eval('function x(){}'); 
6   /* `x` property now references function, and should have no DontDelete */
7   typeof x; // "function" 
8   delete x; // should be `true` 
9   typeof x; // should be "undefined"
遗憾的是，这类欺骗在我尝试中并不总是运行，我可能丢失了一些东西，或者这种行为过于简单不足以引起注意。

浏览器兼容性

从理论上认识事物的工作原理是有用的，但实际影响是至关重要的。当涉及到variable/property creation/deletion时，浏览器遵循标准吗？在大多数是的。

我写了一个简单的测试包检测Global code、Function code 和Eval code代码delete 运算符的兼容性。测试包同时检查 －－ delete运算符的返回值，以及应被删除的属性是否被删除的。delete 运算符返回true或false并不重要，重要的是有DontDelete特性不被删除，反之亦然。

现代浏览器一般都相当兼容，除了这个我早期提到的这个eval特性。下面的浏览器完全通过测试包：Opera 7.54+、Firefox 1.0+、Safari 3.1.2+、Chrome 4+。

Safari 2.x 和3.0.4在删除函数参数时有些问题，这些属性似乎没有创建DontDelete，所以可以删除它们。Safari 2.x 甚至有更多问题，删除非引用（例如delete 1）抛出错误；函数声明创建了可删除属性（但奇怪是变量声明不是），在eval中的变量声明成为不可删除的（但函数声明不是）。

与Safari相似，Konqueror (3.5，但不是 4.3)当删除非引用（例如delete 1）抛出错误，它错误使函数参数可以删除。

Gecko DontDelete bug

Gecko 1.8.x浏览器－－Firefox 2.x、 Camino 1.x、Seamonkey 1.x等显示一个有趣的bug：对一个属性明确地赋值可以删除它的DontDelete特性，即使该属性是通过变量或函数声明来创建的。

代码 
 1 function foo(){} 
 2     delete foo; // false (as expected) 
 3     typeof foo; // "function" (as expected) 
 4   
 5     /* now assign to a property explicitly */
 6   
 7     this.foo = 1; // erroneously clears DontDelete attribute 
 8     delete foo; // true 
 9     typeof foo; // "undefined" 
10   
11     /* note that this doesn't happen when assigning property implicitly */
12   
13     function bar(){} 
14     bar = 1; 
15     delete bar; // false 
16     typeof bar; // "number" (although assignment replaced property)
出乎意料的是，IE5.5 – 8全部通过测试包，删除非引用（例如delete 1）抛出错误（就像在老版的Safari一样）。但事实上有更严重bug存在IE中，这不会立即显现。这些bug都与全局对象相关。

======================================================================


> Written with [StackEdit](https://stackedit.io/).