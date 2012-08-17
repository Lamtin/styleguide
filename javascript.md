#Google JavaScript 编码规范指南

修正版本 2.9

##背景

JavaScript 是一种客户端脚本语言, Google 的许多开源工程中都有用到它. 这份指南列出了编写 JavaScript 时需要遵守的规则.

##JavaScript 语言规范

###变量

声明变量必须加上 `var` 关键字.

- 当你没有写 `var`, 变量就会暴露在全局上下文中, 这样很可能会和现有变量冲突. 另外, 如果没有加上, 很难明确该变量的作用域是什么, 变量也很可能像在局部作用域中, 很轻易地泄漏到 `Document` 或者 `Window` 中, 所以务必用 `var` 去声明变量.

###常量

常量的形式如: `NAMES_LIKE_THIS`, 即使用大写字符, 并用下划线分隔. 你也可用 `@const` 标记来指明它是一个常量. 但请永远不要使用 `const` 关键词.

>对于基本类型的常量, 只需转换命名.

        /**
         * The number of seconds in a minute.
         * @type {number}
         */
        goog.example.SECONDS_IN_A_MINUTE = 60;

>对于非基本类型, 使用 @const 标记.

        /**
         * The number of seconds in each of the given units.
         * @type {Object.<number>}
         * @const
         */
        goog.example.SECONDS_TABLE = {
          minute: 60,
          hour: 60 * 60,
          day: 60 * 60 * 24
        }

>这标记告诉编译器它是常量.

>至于关键词 `const`, 因为 IE 不能识别, 所以不要使用.

###分号

总是使用分号.

>如果仅依靠语句间的隐式分隔, 有时会很麻烦. 你自己更能清楚哪里是语句的起止.

>而且有些情况下, 漏掉分号会很危险:

		// 1.
		MyClass.prototype.myMethod = function() {
		  return 42;
		}  // No semicolon here.
		
		(function() {
		  // Some initialization code wrapped in a function to create a scope for locals.
		})();
		
		
		var x = {
		  'i': 1,
		  'j': 2
		}  // No semicolon here.
		
		// 2.  Trying to do one thing on Internet Explorer and another on Firefox.
		// I know you'd never write code like this, but throw me a bone.
		[normalVersion, ffVersion][isIE]();
		
		
		var THINGS_TO_EAT = [apples, oysters, sprayOnCheese]  // No semicolon here.
		
		// 3. conditional execution a la bash
		-1 == resultOfOperation() || die();

>这段代码会发生些什么诡异事呢?

1. 报 JavaScript 错误 - 例子1上的语句会解释成, 一个函数带一匿名函数作为参数而被调用, 返回`42`后, 又一次被"调用", 这就导致了错误.
2. 例子2中, 你很可能会在运行时遇到 'no such property in undefined' 错误, 原因是代码试图这样 `x[ffVersion][isIE]()` 执行.
3. 当 `resultOfOperation()` 返回非 `NaN` 时, 就会调用`die`, 其结果也会赋给 `THINGS_TO_EAT`.

>为什么?

JavaScript 的语句以分号作为结束符, 除非可以非常准确推断某结束位置才会省略分号. 上面的几个例子产出错误, 均是在语句中声明了函数/对象/数组直接量, 但 闭括号`('}'`或`']')`并不足以表示该语句的结束. 在 JavaScript 中, 只有当语句后的下一个符号是后缀或括号运算符时, 才会认为该语句的结束.

遗漏分号有时会出现很奇怪的结果, 所以确保语句以分号结束.

###嵌套函数

可以使用

>嵌套函数很有用, 比如,减少重复代码, 隐藏帮助函数, 等. 没什么其他需要注意的地方, 随意使用.

###块内函数声明

不要在块内声明一个函数

>不要写成:

		if (x) {
		  function foo() {}
		}

>虽然很多 JS 引擎都支持块内声明函数, 但它不属于 ECMAScript 规范 (见 [ECMA-262](http://www.ecma-international.org/publications/standards/Ecma-262.htm), 第13和14条). 各个浏览器糟糕的实现相互不兼容, 有些也与未来 ECMAScript 草案相违背. ECMAScript 只允许在脚本的根语句或函数中声明函数. 如果确实需要在块中定义函数, 建议使用函数表达式来初始化变量:

		if (x) {
		  var foo = function() {}
		}

###异常

可以

>你在写一个比较复杂的应用时, 不可能完全避免不会发生任何异常. 大胆去用吧.

###自定义异常

可以

>有时发生异常了, 但返回的错误信息比较奇怪, 也不易读. 虽然可以将含错误信息的引用对象或者可能产生错误的完整对象传递过来, 但这样做都不是很好, 最好还是自定义异常类, 其实这些基本上都是最原始的异常处理技巧. 所以在适当的时候使用自定义异常.

###标准特性

总是优于非标准特性.

>最大化可移植性和兼容性, 尽量使用标准方法而不是用非标准方法, (比如, 优先用`string.charAt(3)` 而不用 `string[3]` , 通过 DOM 原生函数访问元素, 而不是使用应用封装好的快速接口.

###封装基本类型

不要

>没有任何理由去封装基本类型, 另外还存在一些风险:

		var x = new Boolean(false);
		if (x) {
		  alert('hi');  // Shows 'hi'.
		}

>除非明确用于类型转换, 其他情况请千万不要这样做！

		var x = Boolean(0);
		if (x) {
		  alert('hi');  // This will never be alerted.
		}
		typeof Boolean(0) == 'boolean';
		typeof new Boolean(0) == 'object';

>有时用作 `number`, `string` 或 `boolean`时, 类型的转换会非常实用.

###多级原型结构

不是首选

>多级原型结构是指 JavaScript 中的继承关系. 当你自定义一个D类, 且把B类作为其原型, 那么这就获得了一个多级原型结构. 这些原型结构会变得越来越复杂!

>使用 the Closure 库 中的 `goog.inherits()` 或其他类似的用于继承的函数, 会是更好的选择.

		function D() {
		  goog.base(this)
		}
		goog.inherits(D, B);
		
		D.prototype.method = function() {
		  ...
		};

###方法定义

`Foo.prototype.bar = function() { ... };`

>有很多方法可以给构造器添加方法或成员, 我们更倾向于使用如下的形式:

		Foo.prototype.bar = function() {
		  /* ... */
		};

###闭包

可以, 但小心使用.

>闭包也许是 JS 中最有用的特性了. 有一份比较好的介绍闭包原理的文档.

>有一点需要牢记, 闭包保留了一个指向它封闭作用域的指针, 所以, 在给 DOM 元素附加闭包时, 很可能会产生循环引用, 进一步导致内存泄漏. 比如下面的代码:

		function foo(element, a, b) {
		  element.onclick = function() { /* uses a and b */ };
		}

>这里, 即使没有使用 `element`, 闭包也保留了 `element`, `a` 和 `b` 的引用, . 由于 `element` 也保留了对闭包的引用, 这就产生了循环引用, 这就不能被 GC 回收. 这种情况下, 可将代码重构为:

		function foo(element, a, b) {
		  element.onclick = bar(a, b);
		}
		
		function bar(a, b) {
		  return function() { /* uses a and b */ }
		}

###eval()

只用于解析序列化串 (如: 解析 RPC 响应)

>`eval()` 会让程序执行的比较混乱, 当 `eval()` 里面包含用户输入的话就更加危险. 可以用其他更佳的, 更清晰, 更安全的方式写你的代码, 所以一般情况下请不要使用 `eval()`. 当碰到一些需要解析序列化串的情况下(如, 计算 RPC 响应), 使用 `eval` 很容易实现.

>解析序列化串是指将字节流转换成内存中的数据结构. 比如, 你可能会将一个对象输出成文件形式:

		users = [
		  {
		    name: 'Eric',
		    id: 37824,
		    email: 'jellyvore@myway.com'
		  },
		  {
		    name: 'xtof',
		    id: 31337,
		    email: 'b4d455h4x0r@google.com'
		  },
		  ...
		];

>很简单地调用 `eval` 后, 把表示成文件的数据读取回内存中.

>类似的, `eval()` 对 RPC 响应值进行解码. 例如, 你在使用 XMLHttpRequest 发出一个 RPC 请求后, 通过 `eval()` 将服务端的响应文本转成 JavaScript 对象:

		var userOnline = false;
		var user = 'nusrat';
		var xmlhttp = new XMLHttpRequest();
		xmlhttp.open('GET', 'http://chat.google.com/isUserOnline?user=' + user, false);
		xmlhttp.send('');
		// Server returns:
		// userOnline = true;
		if (xmlhttp.status == 200) {
		  eval(xmlhttp.responseText);
		}
		// userOnline is now true.
		
###with() {}

不要使用

>使用 with 让你的代码在语义上变得不清晰. 因为 with 的对象, 可能会与局部变量产生冲突, 从而改变你程序原本的用义. 下面的代码是干嘛的?

		with (foo) {
		  var x = 3;
		  return x;
		}

>答案: 任何事. 局部变量 x 可能被 foo 的属性覆盖, 当它定义一个 setter 时, 在赋值 3 后会执行很多其他代码. 所以不要使用 with 语句.

###this

仅在对象构造器, 方法, 闭包中使用.

>`this` 的语义很特别. 有时它引用一个全局对象(大多数情况下), 调用者的作用域(使用 `eval`时), DOM 树中的节点(添加事件处理函数时), 新创建的对象(使用一个构造器), 或者其他对象(如果函数被 `call()` 或 `apply()`).

>使用时很容易出错, 所以只有在下面两个情况时才能使用:

1. 在构造器中
2. 对象的方法(包括创建的闭包)中

###for-in 循环

只用于 object/map/hash 的遍历

>对 `Array` 用 for-in 循环有时会出错. 因为它并不是从 0 到 length - 1 进行遍历, 而是所有出现在对象及其原型链的键值. 下面就是一些失败的使用案例:

		function printArray(arr) {
		  for (var key in arr) {
		    print(arr[key]);
		  }
		}

		printArray([0,1,2,3]);  // This works.
		
		var a = new Array(10);
		printArray(a);  // This is wrong.
		
		a = document.getElementsByTagName('*');
		printArray(a);  // This is wrong.
		
		a = [0,1,2,3];
		a.buhu = 'wine';
		printArray(a);  // This is wrong again.
		
		a = new Array;
		a[3] = 3;
		printArray(a);  // This is wrong again.

>而遍历数组通常用最普通的 `for` 循环.

		function printArray(arr) {
		  var l = arr.length;
		  for (var i = 0; i < l; i++) {
		    print(arr[i]);
		  }
		}

###关联数组

永远不要使用 `Array` 作为 map/hash/associative 数组.

>数组中不允许使用非整型作为索引值, 所以也就不允许用关联数组. 而取代它使用 Object 来表示 map/hash 对象. `Array` 仅仅是扩展自 `Object` (类似于其他 JS 中的对象, 就像 `Date`, `RegExp` 和 `String`)一样来使用.

###多行字符串

不要使用

>不要这样写长字符串:

		var myString = 'A rather long string of English text, an error message \
		                actually that just keeps going and going -- an error \
		                message to make the Energizer bunny blush (right through \
		                those Schwarzenegger shades)! Where was I? Oh yes, \
		                you\'ve got an error and all the extraneous whitespace is \
		                just gravy.  Have a nice day.';

>在编译时, 不能忽略行起始位置的空白字符; "\" 后的空白字符会产生奇怪的错误; 虽然大多数脚本引擎支持这种写法, 但它不是 ECMAScript 的标准规范.

###Array 和 Object 直接量

使用

>使用 `Array` 和 `Object` 语法, 而不使用 `Array` 和 `Object` 构造器.

>使用 Array 构造器很容易因为传参不恰当导致错误.

		// Length is 3.
		var a1 = new Array(x1, x2, x3);
		
		// Length is 2.
		var a2 = new Array(x1, x2);
		
		// If x1 is a number and it is a natural number the length will be x1.
		// If x1 is a number but not a natural number this will throw an exception.
		// Otherwise the array will have one element with x1 as its value.
		var a3 = new Array(x1);
		
		// Length is 0.
		var a4 = new Array();

>如果传入一个参数而不是2个参数, 数组的长度很有可能就不是你期望的数值了.

>为了避免这些歧义, 我们应该使用更易读的直接量来声明.

		var a = [x1, x2, x3];
		var a2 = [x1, x2];
		var a3 = [x1];
		var a4 = [];

>虽然 `Object` 构造器没有上述类似的问题, 但鉴于可读性和一致性考虑, 最好还是在字面上更清晰地指明.

		var o = new Object();
		
		var o2 = new Object();
		o2.a = 0;
		o2.b = 1;
		o2.c = 2;
		o2['strange key'] = 3;

>应该写成:

		var o = {};
		
		var o2 = {
		  a: 0,
		  b: 1,
		  c: 2,
		  'strange key': 3
		};

###修改内置对象的原型

不要

>千万不要修改内置对象, 如 `Object.prototype` 和 `Array.prototype` 的原型. 而修改内置对象, 如 `Function.prototype` 的原型, 虽然少危险些, 但仍会导致调试时的诡异现象. 所以也要避免修改其原型.

###IE下的条件注释

不要使用

>不要这样子写:

		var f = function () {
		    /*@cc_on if (@_jscript) { return 2* @*/  3; /*@ } @*/
		};

>条件注释妨碍自动化工具的执行, 因为在运行时, 它们会改变 JavaScript 语法树.

##JavaScript 编码风格

###命名

>通常, 使用 `functionNamesLikeThis`, `variableNamesLikeThis`, `ClassNamesLikeThis`, `EnumNamesLikeThis`, `methodNamesLikeThis`, 和 `SYMBOLIC_CONSTANTS_LIKE_THIS`.

####属性和方法

>文件或类中的 私有 属性, 变量和方法名应该以下划线 "_" 开头.
保护 属性, 变量和方法名不需要下划线开头, 和公共变量名一样.
更多有关 私有 和 保护的信息见, [visibility](http://docs.kissyui.com/source/tutorials/styleguide/google/javascriptguide.xml#Visibility__private_and_protected_fields_).

####方法和函数参数

>可选参数以 `opt_` 开头.

>函数的参数个数不固定时, 应该添加最后一个参数 `var_args` 为参数的个数. 你也可以不设置 `var_args`而取代使用 `arguments`.

>可选和可变参数应该在 `@param` 标记中说明清楚. 虽然这两个规定对编译器没有任何影响, 但还是请尽量遵守

####Getters 和 Setters

>Getters 和 setters 并不是必要的. 但只要使用它们了, 就请将 getters 命名成 `getFoo()` 形式, 将 setters 命名成 `setFoo(value)` 形式. (对于布尔类型的 getters, 使用 `isFoo()` 也可.)

####命名空间

>JavaScript 不支持包和命名空间.

>不容易发现和调试全局命名的冲突, 多个系统集成时还可能因为命名冲突导致很严重的问题. 为了提高 JavaScript 代码复用率, 我们遵循下面的约定以避免冲突.

#####为全局代码使用命名空间

>在全局作用域上, 使用一个唯一的, 与工程/库相关的名字作为前缀标识. 比如, 你的工程是 "Project Sloth", 那么命名空间前缀可取为 sloth.*.

		var sloth = {};
		
		sloth.sleep = function() {
		  ...
		};

>许多 JavaScript 库, 包括 the Closure Library and Dojo toolkit 为你提供了声明你自己的命名空间的函数. 比如:

		goog.provide('sloth');
		
		sloth.sleep = function() {
		  ...
		};

#####明确命名空间所有权

>当选择了一个子命名空间, 请确保父命名空间的负责人知道你在用哪个子命名空间, 比如说, 你为工程 'sloths' 创建一个 'hats' 子命名空间, 那确保 Sloth 团队人员知道你在使用 sloth.hats.

#####外部代码和内部代码使用不同的命名空间

>"外部代码" 是指来自于你代码体系的外部, 可以独立编译. 内外部命名应该严格保持独立. 如果你使用了外部库, 他的所有对象都在 foo.hats.* 下, 那么你自己的代码不能在 foo.hats.*下命名, 因为很有可能其他团队也在其中命名.

		foo.require('foo.hats');
		
		/**
		 * WRONG -- Do NOT do this.
		 * @constructor
		 * @extend {foo.hats.RoundHat}
		 */
		foo.hats.BowlerHat = function() {
		};

>如果你需要在外部命名空间中定义新的 API, 那么你应该直接导出一份外部库, 然后在这份代码中修改. 在你的内部代码中, 应该通过他们的内部名字来调用内部 API , 这样保持一致性可让编译器更好的优化你的代码.

		foo.provide('googleyhats.BowlerHat');
		
		foo.require('foo.hats');
		
		/**
		 * @constructor
		 * @extend {foo.hats.RoundHat}
		 */
		googleyhats.BowlerHat = function() {
		  ...
		};

		goog.exportSymbol('foo.hats.BowlerHat', googleyhats.BowlerHat);

#####重命名那些名字很长的变量, 提高可读性

>主要是为了提高可读性. 局部空间中的变量别名只需要取原名字的最后部分.

		/**
		 * @constructor
		 */
		some.long.namespace.MyClass = function() {
		};
		
		/**
		 * @param {some.long.namespace.MyClass} a
		 */
		some.long.namespace.MyClass.staticHelper = function(a) {
		  ...
		};
		
		myapp.main = function() {
		  var MyClass = some.long.namespace.MyClass;
		  var staticHelper = some.long.namespace.MyClass.staticHelper;
		  staticHelper(new MyClass());
		};

>不要对命名空间创建别名.

		myapp.main = function() {
		  var namespace = some.long.namespace;
		  namespace.MyClass.staticHelper(new namespace.MyClass());
		};

>除非是枚举类型, 不然不要访问别名变量的属性.

		/** @enum {string} */
		some.long.namespace.Fruit = {
		  APPLE: 'a',
		  BANANA: 'b'
		};
		
		myapp.main = function() {
		  var Fruit = some.long.namespace.Fruit;
		  switch (fruit) {
		    case Fruit.APPLE:
		      ...
		    case Fruit.BANANA:
		      ...
		  }
		};
		myapp.main = function() {
		  var MyClass = some.long.namespace.MyClass;
		  MyClass.staticHelper(null);
		};

>不要在全局范围内创建别名, 而仅在函数块作用域中使用.

###文件名

>文件名应该使用小写字符, 以避免在有些系统平台上不识别大小写的命名方式. 文件名以.js结尾, 不要包含除 - 和 _ 外的标点符号(使用 - 优于 _).

###自定义 toString() 方法

应该总是成功调用且不要抛异常.

>可自定义 `toString()` 方法, 但确保你的实现方法满足: (1) 总是成功 (2) 没有其他负面影响. 如果不满足这两个条件, 那么可能会导致严重的问题, 比如, 如果 `toString()` 调用了包含 `assert` 的函数, `assert` 输出导致失败的对象, 这在 `toString()` 也会被调用.

###延迟初始化

可以

>没必要在每次声明变量时就将其初始化.

###明确作用域

任何时候都需要

>任何时候都要明确作用域 - 提高可移植性和清晰度. 例如, 不要依赖于作用域链中的 window 对象. 可能在其他应用中, 你函数中的 window 不是指之前的那个窗口对象.

###代码格式化

展开见详细描述.

>主要依照C++ 格式规范 ( 中文版 ), 针对 JavaScript, 还有下面一些附加说明.

####大括号

>分号会被隐式插入到代码中, 所以你务必在同一行上插入大括号. 例如:

		if (something) {
		  // ...
		} else {
		  // ...
		}

####数组和对象的初始化

>如果初始值不是很长, 就保持写在单行上:

		var arr = [1, 2, 3];  // No space after [ or before ].
		var obj = {a: 1, b: 2, c: 3};  // No space after { or before }.

>初始值占用多行时, 缩进2个空格.

		// Object initializer.
		var inset = {
		  top: 10,
		  right: 20,
		  bottom: 15,
		  left: 12
		};
		
		// Array initializer.
		this.rows_ = [
		  '"Slartibartfast" <fjordmaster@magrathea.com>',
		  '"Zaphod Beeblebrox" <theprez@universe.gov>',
		  '"Ford Prefect" <ford@theguide.com>',
		  '"Arthur Dent" <has.no.tea@gmail.com>',
		  '"Marvin the Paranoid Android" <marv@googlemail.com>',
		  'the.mice@magrathea.com'
		];
		
		// Used in a method call.
		goog.dom.createDom(goog.dom.TagName.DIV, {
		  id: 'foo',
		  className: 'some-css-class',
		  style: 'display:none'
		}, 'Hello, world!');

>比较长的标识符或者数值, 不要为了让代码好看些而手工对齐. 如:

		CORRECT_Object.prototype = {
		  a: 0,
		  b: 1,
		  lengthyName: 2
		};

>不要这样做:

		WRONG_Object.prototype = {
		  a          : 0,
		  b          : 1,
		  lengthyName: 2
		};

####函数参数

>尽量让函数参数在同一行上. 如果一行超过 80 字符, 每个参数独占一行, 并以4个空格缩进, 或者与括号对齐, 以提高可读性. 尽可能不要让每行超过80个字符. 比如下面这样:

		// Four-space, wrap at 80.  Works with very long function names, survives
		// renaming without reindenting, low on space.
		goog.foo.bar.doThingThatIsVeryDifficultToExplain = function(
		    veryDescriptiveArgumentNumberOne, veryDescriptiveArgumentTwo,
		    tableModelEventHandlerProxy, artichokeDescriptorAdapterIterator) {
		  // ...
		};
		
		// Four-space, one argument per line.  Works with long function names,
		// survives renaming, and emphasizes each argument.
		goog.foo.bar.doThingThatIsVeryDifficultToExplain = function(
		    veryDescriptiveArgumentNumberOne,
		    veryDescriptiveArgumentTwo,
		    tableModelEventHandlerProxy,
		    artichokeDescriptorAdapterIterator) {
		  // ...
		};
		
		// Parenthesis-aligned indentation, wrap at 80.  Visually groups arguments,
		// low on space.
		function foo(veryDescriptiveArgumentNumberOne, veryDescriptiveArgumentTwo,
		             tableModelEventHandlerProxy, artichokeDescriptorAdapterIterator) {
		  // ...
		}
		
		// Parenthesis-aligned, one argument per line.  Visually groups and
		// emphasizes each individual argument.
		function bar(veryDescriptiveArgumentNumberOne,
		             veryDescriptiveArgumentTwo,
		             tableModelEventHandlerProxy,
		             artichokeDescriptorAdapterIterator) {
		  // ...
		}

####传递匿名函数

>如果参数中有匿名函数, 函数体从调用该函数的左边开始缩进2个空格, 而不是从 function 这个关键字开始. 这让匿名函数更加易读 (不要增加很多没必要的缩进让函数体显示在屏幕的右侧).

		var names = items.map(function(item) {
		                        return item.name;
		                      });
		
		prefix.something.reallyLongFunctionName('whatever', function(a1, a2) {
		  if (a1.equals(a2)) {
		    someOtherLongFunctionName(a1);
		  } else {
		    andNowForSomethingCompletelyDifferent(a2.parrot);
		  }
		});

####更多的缩进

>事实上, 除了 初始化数组和对象 , 和传递匿名函数外, 所有被拆开的多行文本要么选择与之前的表达式左对齐, 要么以4个(而不是2个)空格作为一缩进层次.

		someWonderfulHtml = '' +
		                    getEvenMoreHtml(someReallyInterestingValues, moreValues,
		                                    evenMoreParams, 'a duck', true, 72,
		                                    slightlyMoreMonkeys(0xfff)) +
		                    '';
		
		thisIsAVeryLongVariableName =
		    hereIsAnEvenLongerOtherFunctionNameThatWillNotFitOnPrevLine();
		
		thisIsAVeryLongVariableName = 'expressionPartOne' + someMethodThatIsLong() +
		    thisIsAnEvenLongerOtherFunctionNameThatCannotBeIndentedMore();
		
		someValue = this.foo(
		    shortArg,
		    'Some really long string arg - this is a pretty common case, actually.',
		    shorty2,
		    this.bar());
		
		if (searchableCollection(allYourStuff).contains(theStuffYouWant) &&
		    !ambientNotification.isActive() && (client.isAmbientSupported() ||
		                                        client.alwaysTryAmbientAnyways()) {
		  ambientNotification.activate();
		}

####空行

>使用空行来划分一组逻辑上相关联的代码片段.

		doSomethingTo(x);
		doSomethingElseTo(x);
		andThen(x);
		
		nowDoSomethingWith(y);
		
		andNowWith(z);

####二元和三元操作符

>操作符始终跟随着前行, 这样就不用顾虑分号的隐式插入问题. 如果一行实在放不下, 还是按照上述的缩进风格来换行.

		var x = a ? b : c;  // All on one line if it will fit.
		
		// Indentation +4 is OK.
		var y = a ?
		    longButSimpleOperandB : longButSimpleOperandC;
		
		// Indenting to the line position of the first operand is also OK.
		var z = a ?
		        moreComplicatedB :
		        moreComplicatedC;

###括号

只在需要的时候使用

>不要滥用括号, 只在必要的时候使用它.

>对于一元操作符(如`delete`, `typeof` 和 `void` ), 或是在某些关键词(如 `return`, `throw`, `case`, `new` )之后, 不要使用括号.

###字符串

使用 ' 优于 "

>单引号 (') 优于双引号 ("). 当你创建一个包含 HTML 代码的字符串时就知道它的好处了.

		var msg = 'This is some HTML';

###可见性 (私有域和保护域)

>推荐使用 JSDoc 中的两个标记: `@private` 和 `@protected`
JSDoc 的两个标记 `@private` 和 `@protected` 用来指明类, 函数, 属性的可见性域.

>标记为 `@private` 的全局变量和函数, 表示它们只能在当前文件中访问.

>标记为 `@private` 的构造器, 表示该类只能在当前文件或是其静态/普通成员中实例化; 私有构造器的公共静态属性在当前文件的任何地方都可访问, 通过 `instanceof` 操作符也可.

>永远不要为 全局变量, 函数, 构造器加 `@protected` 标记.

		// File 1.
		// AA_PrivateClass_ and AA_init_ are accessible because they are global
		// and in the same file.
		
		/**
		 * @private
		 * @constructor
		 */
		AA_PrivateClass_ = function() {
		};
		
		/** @private */
		function AA_init_() {
		  return new AA_PrivateClass_();
		}

		AA_init_();

>标记为 `@private` 的属性, 在当前文件中可访问它; 如果是类属性私有, "拥有"该属性的类的所有静态/普通成员也可访问, 但它们不能被不同文件中的子类访问或覆盖.

>标记为 `@protected` 的属性, 在当前文件中可访问它, 如果是类属性保护, 那么"拥有"该属性的类及其子类中的所有静态/普通成员也可访问.

>注意: 这与 C++, Java 中的私有和保护不同, 它们是在当前文件中, 检查是否具有访问私有/保护属性的权限, 有权限即可访问, 而不是只能在同一个类或类层次上. 而 C++ 中的私有属性不能被子类覆盖. (C++/Java 中的私有/保护是指作用域上的可访问性, 在可访问性上的限制. JS 中是在限制在作用域上. PS: 可见性是与作用域对应)

		// File 1.
		
		/** @constructor */
		  AA_PublicClass = function() {
		};
		
		/** @private */
		AA_PublicClass.staticPrivateProp_ = 1;
		
		/** @private */
		AA_PublicClass.prototype.privateProp_ = 2;
		
		/** @protected */
		AA_PublicClass.staticProtectedProp = 31;
		
		/** @protected */
		AA_PublicClass.prototype.protectedProp = 4;
		
		// File 2.
		
		/**
		 * @return {number} The number of ducks we've arranged in a row.
		 */
		AA_PublicClass.prototype.method = function() {
		  // Legal accesses of these two properties.
		  return this.privateProp_ + AA_PublicClass.staticPrivateProp_;
		};
		
		// File 3.
		
		/**
		 * @constructor
		 * @extends {AA_PublicClass}
		 */
		AA_SubClass = function() {
		  // Legal access of a protected static property.
		  AA_PublicClass.staticProtectedProp = this.method();
		};
		goog.inherits(AA_SubClass, AA_PublicClass);
		
		/**
		 * @return {number} The number of ducks we've arranged in a row.
		 */
		AA_SubClass.prototype.method = function() {
		  // Legal access of a protected instance property.
		  return this.protectedProp;
		};

