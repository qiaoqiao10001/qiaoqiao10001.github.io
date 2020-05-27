---
title: JavaScript底层运行原理
date: 2020-05-28 00:19:35
tags: 學習 JavaScript
---


基本数据类型

```javascript
    null undefined number string boolean
    //typeof 返回值类型
    typeof null == 'object' 
    typeof undefined === 'undefined'
    typeof 12 === 'number'
    typeof '123' === 'string'
    typeof true === 'boolean'
//	Symbol BigInt
```

引用数据类型：

```javascript
    // object function 
    typeof [123] === 'object'
    typeof {} === 'object'
    typeof function(){} === 'function'
```

## 堆、栈内存

- alert出来的结果都要toString()

```javascrip
    let a = 0;
    let b = a;
    b++
    alert(a)  //==> '0'

```

- 浏览器渲染的时候(编译器)
  - 词法解析
  - AST抽象语法树
  - 构建能够执行的代码
  - 引 擎（v8）
  - 变量提升、作用域，闭包 、堆栈内存。
  - GO、VO、AO、EC、ECStack
- 栈内存
  - 存储基本类型的值、执行代码
- 基本类型变量声明
  - 创建变量
  - 创建值
  - 把值和变量关联
  - 变量和值的关联就是指针概念
  - const 声明的是指针不可变的变量
- 引用类型变量
  - 引用类型是复杂的，所以特殊处理
  - 首先会开辟一个存储对象键值对的内存空间（堆内存）
  - 堆内存中有一段可被后续查找的地址（16进制）
  - 在栈内存中，把堆内存地址给了声明的变量
- null、undefined
  - a = null 是把变量指针指向空指针、解除这个变量的引用。赋值，暂时不想赋值，后期要赋值，则赋值null
  - undefined 声明了。
- EC （执行上下文）
  - 某个域下面的代码执行都胡形成自己的执行上下文
  - 全局 EC(G) 全局
  - EC 某个函数
  - 执行时候，进栈，执行完之后，出栈
  - 有的还要用，比如闭包，会把其压入栈底，以便再次使用
- GO (全局对象)： Global Object、存储浏览器api对象 
  - 浏览器端会把变量赋值给window
- VO(G) 全局变量对象： 全局上下文存储全局变量的空间，不是GO

## 执行上下文

- JS代码首次运行,都会先创建一个全局*执行上下文*并压入到执行栈中,之后每当有函数被调用,都会创建一个新的函数*执行上下文*并压入栈内;
- 每当一个函数被调用时都会创建一个函数上下文；需要注意的是，同一个函数被多次调用，都会创建一个新的上下文。

## 作用域

- 创建函数的时候就形成了作用域，如果全局上下文当中创建一个函数，当前函数的作用域就是全局上下文，在哪创建的函数，他的作用域就是此时的上下文

- 每个函数执行的时候，里头都是该函数私有的执行上下文

- ```javascript
  function A(){
  	function B(){}
  }
  A();
  //B的执行上下文就就是A的作用域
  ```



## 函数声明&执行

1. 函数执行时候，形成自己的私有执行上下文，供代码执行，然后放到栈顶进行执行
2. 函数声明的时候，就会有自己的作用域，如果全局声明那么其作用域是EC(G)，全局执行上下文；其次， 函数会有自己的私有变量对象 也就是AO(FN) ；还会初始化作用域链 (scope chain) <EC(FN)  ，EC(G)>; 所以访问函数内的变量会首先找函数私有作用域中的变量对象，然后通过作用域去查找其执行环境中的标识符；  函数上下文取决于函数在哪创建的
3. 初始化this  arguments  形参赋值



### 函数创建的时候发生的()

- 创建一个堆空间（存储代码字符串和对应的键值对）
- 初始化了当前函数的作用域
  - [[scope]] = 所在上下文EC中的变量对象VO/AO

### 函数执行的时候

- 创建一个新的执行上下文EC(压缩到栈ECStack中执行)
- 初始化this指向
- 初始化作用域链 [[scope-chain]]
- 创建AO活动对象来存储变量
  - Arguments ，形参赋值 ==> 如果有变量提升  ==> 代码执行
- 默认返回undefined



### 自执行函数

- 本身应该是匿名函数，但是一般都具名化了

- 特点

  - 这个名字只能在函数内部使用，函数外用不了
  - 这个名字的变量，在函数内部值是不能被修改的(改了也没用 )

  ```js
  var b = 10
  (function b(){
    b = 20; //匿名函数值不能被修改，改了也没用
    console.log(b) // 这个是打印给函数本身
  })()
  console.log(b)
  // arguments.callee代表函数本身
  
  // 但是如果是let function var等操作, 会改成正常变量
  var b = 10
  (function AAA(){
    let AAA = 20; // 
    console.log(AAA) // 20
  })()
  console.log(b)
  ```

  



## 作用域链、闭包、this

- this指向 理解为执行主体

1. 函数执行，看前面有无点运算符 入 obj.fn() this指向obj
2. 给元素绑定时间方法（dom0 、dom2级）事件触发，this指向当前元素本身

```javascript
    var x = 10;
    var obj = {
        x: 20,
        fn:function(){
            console.log(x)
        }
    }
    
    var fn = obj.fn();
    fn();  // 10
    obj.fn(); // 20
    var box = document.getElementById('box')
    box.x = 30;
    box.onclick = function(){
      // this => box
        obj.fn(); // => 20
    }
    
    box.onclick = obj.fn; // => 30
```

### 变量提升

- 代码执行之前，会把var 或者function关键字的声明或者定义（带var的只提前声明，带function的会提前声明+定义【赋值】）

- {}中出现let/const/function，才会形成块级作用域

- ```js
  {
    function foo(){}  
    foo = 1
    function foo(){}
  }
  console.log(foo)
  
  ```

### 闭包

- 函数执行会**形成全新的私有上下文**，这个上下文可能被释放，也可能不被释放，不论是否被释放，它的作用是：

1. 保护：划分一个独立的代码执行区域，在这个区域中有自己私有变量存储的空间，而用到的私有变量和其它区域中的变量不会有任何的冲突（防止全局变量污染）

2. 保存：如果上下文不被销毁，那么存储的私有变量的值也不会被销毁，可以被其下级上下文中调取使用

   ps:我们把函数执行，形成私有上下文，来保存和保护私有变量的机制，称之为“闭包”  =>它是一种机制

### 闭包理解

- 市面上一般认为只有形成的私有上下文不被释放，才算是闭包（因为如果一但释放，之前的东西也就不存在了）；还有人认为，只有一下级上下文用到了此上下文中的动西才算闭包；



```js
// 建议：浏览器加载页面会把代码放到栈内存中执行（也就是ECStack），函数进栈执行会产生一个私有的上下文（也就是EC），此上下文能保护里面的私有变量（也就是AO）不受外界的干扰，并且如果当前上下文中的某些内容，被上下文以外的内容所占用，当前上下文是不会出栈释放的，这样可以保存里面的变量和变量值，所以我认为闭包是一种保存和保护内部私有变量的机制...在真实的项目中，其实我应用闭包的场景还是很多的，例如：
// 1. 我会基于闭包把自己编写的模块内容包起来，这样自己编写的代码都是私有的，防止和全局变量或者别人的代码冲突，这一点利用的是闭包的保护机制
// 2. 在没有用LET之前，我们循环处理事件绑定，在事件触发需要用到索引值的时候，我们基于闭包，把每一轮循环的索引值保存起来，这样来实现我们的需求，只不过现在都是基于LET来完成，因为LET会产生块级作用域来保存需要的内容（机制和闭包类似）
// 但是不建议过多使用闭包，因为形成不被释放的上下文，是占用栈内存空间的，过多使用会导致页面渲染变慢，所以要合理应用闭包
// 除了这些传统的业务开发中会应用闭包，我之前在研究别人源码和自己写一写插件的时候，往往会利用一些JS高阶编程技巧来实现代码的管理和功能的开发，他们的底层机制其实就是闭包，例如：
// 1. 惰性函数
// 2. 柯理化函数
// 3. compose函数
// ......

```





### let var const 

- var 声明的会给全局对象GO增加一个对应的属性；
- 但是let变量不会给全局增加变量
- let 不能重复声明
- 不带var，let,const声明的变量会在代码解析阶段给当前环境声明一个属性

```javascript
    let x =1;
    let x=2; //Uncaught SyntaxError: Identifier 'x' has already been declared
```

- let const 声明
  - let 声明的变量是可以更改指针指向的，可以重新赋值；
  - const声明的变量是不允许改变指针指向的



### 函数执行和new 函数执行

- 创建一个新的执行上下文EC(压缩到栈ECStack中执行)

- 初始化this指向

- 初始化作用域链 [[scope-chain]]

- 创建AO活动对象来存储变量

  - Arguments ，形参赋值 ==> 如果有变量提升  ==> 代码执行

   new 执行不同的地方

  - 【new】默认创建一个对象，而这个对象就是当前类的实例
  - 【new】声明this指向，指向新创建的实例
  - 【new】不论是否有return语句，默认返回新创建的实例，如果返回的引用类型的值则覆盖默认实例

### new执行的时候的步骤

- 默认创建一个实例对象（当前类的实例）
- 也会把类当做普通函数执行，接受这个函数的返回结果
- 执行的时候保证函数中的this指向创建的实例
- 如果自己返回引用值，则返回自己的引用值，否则返回创建的实例

```javascript
    function _new(fn,...args){
        // let obj = {};
        // obj.__proto__ = fn.prototype;
        // 创建一个空对象让其__proto__属性指向fn的prototype
        let obj = Object.create(fn.prototype)
        let result = fn.call(obj, ...args);
        if((typeof result !==null && typeof result === 'object') || typeof result ==='function'){
            return result;
        }
        return obj;
    }
    
    function Dog(name){
        this.name = name
    }
    Dog.prototype.bark = function(){
        console.log('汪汪汪')
    }
    Dog.prototype.sayName = function(){
        console.log('my name is '+ this.name)
    }
    let sanmao = _new(Dog,'三毛');
```



### 变量赋值都是指针

```javascript
fn();
function fn(){ console.log(1); }
fn();
function fn(){ console.log(2); }
fn();
var fn = function(){ console.log(3); }  // 这里开始重新赋值之后才执行
fn();
function fn(){ console.log(4); }
fn();
function fn(){ console.log(5); }
fn();
```

#### 执行过程分析  function fn(){}  （声明fn，指向一个地址）

fn是指向一个地址，这时不断改变fn的指针指向

代码执行fn



- 





MVVM

proxy不用递归数据，加强性能



重写数组方法，

对对象和数组分别观测

# 对象做属性名的时候转为"[object Object]"

页面渲染



如果传入了el则需要实现挂载流程；

写的是模板就要进行编译，render就不用编译

render函数 把模板编译成对象

ast语法树是用对象来描述原生语法的，

虚拟dom是对象描述dom节点

nodeType: 

nodeType 属性返回以数字值返回指定节点的节点类型。

如果节点是元素节点，则 nodeType 属性将返回 1。

如果节点是属性节点，则 nodeType 属性将返回 2。

```html
<div id="#app">
  <p>
    heelo
  </p>
</div>
```

```javascript
// ast语法树
let root = {
  tag: 'div',
  attrs: [{name:'id',value: 'app'}],
  parent: null,
  children: {
    tag: 'p',
    attr: [],
    parent: root,
    type: 1,  //nodeType
    children:[
      {
        
      }
    ]
  }
}
```

ast生成最终的render函数（就是模板引擎）

先把html转成ast语法树，然后再转成字符串

模板编译->AST语法树->render()->VNode->DOM-Diff



### 惰性函数：只会在执行的第一个分支调用函数

```js
function addEvent (type, element, fun) {
  // 只检测了了一次能力，来返回这个函数
  if (element.addEventListener) {
    addEvent = function (type, element, fun) {
      element.addEventListener(type, fun, false);
    }
  }
  else if(element.attachEvent){
    addEvent = function (type, element, fun) {
      element.attachEvent('on' + type, fun);
    }
  }
  else{
    addEvent = function (type, element, fun) {
      element['on' + type] = fun;
    }
  }
  return addEvent(type, element, fun);
}

```



### 柯里化函数： 将多个参数的函数转换为单一参数

```js
// 正常正则验证字符串 reg.test(txt)

// 函数封装后
function check(reg, txt) {
    return reg.test(txt)
}

check(/\d+/g, 'test')       //false
check(/[a-z]+/g, 'test')    //true

// Currying后
function curryingCheck(reg) {
    return function(txt) {
        return reg.test(txt)
    }
}

var hasNumber = curryingCheck(/\d+/g)
var hasLetter = curryingCheck(/[a-z]+/g)

hasNumber('test1')      // true
hasNumber('testtest')   // false
hasLetter('21212')      // false
```



### 深拷贝



```js
var deepClone = (target) => {
    if(typeof target ==='object' &&target !== null){
        const cloneTarget = Array.isArray(target) ? [] : {};
        for(let prop in target){
            if(target.hasOwnProperty(prop)){
                cloneTarget[prop] = deepClone(target[prop])
            }
        }
    		return cloneTarget;
    }else{
      return target;
    }
}
```



## 类型转换规律小结

**在加号两边出现字符串（或者对象）的情况下，加号一定是字符串拼接** ==>  <u>对象本身是要转数字进行计算的，转为数字的过程中需要先转字符串，因为字符串所以一定先拼接</u>

null 转数字是0

undefined转数字是NaN

**对象转字符串先调用valueOf获取原始值（一般是基本类型值），<u>如果不是原始值</u>则继续调用toString();**

==规则

1. 对象==字符串   对象转换为字符串
   [10] == '10'    true

2. null == undefined  （三个等号下不相等），但是和其它任何的值都不相等
   0 == null   false

3. NaN和谁（包括自己）都不相等

4. 剩下的情况都是转换为数字在做比较的

5. ![]==false
   运算符优先级   ![]   再算比较
   ![]  转换为布尔值进行取反（把其它类型转换为布尔类型遵循的规律： 只有 0/NaN/null/undefined/'' 五个值是false，其余的都是true）  => false

   false == false    true

6. +值是数学操作符  和+类似  {}出现在+前面的时候{}是代码块  {}+0 是把前面当做一个代码块

7. 0+{} 是数学运算了 得到 0[object Object] 

   大括号在运算符前面

   1在没有使用小括号处理优先级的情况下  不认为是数学运算，加小括号才算

   2出现在运算符的后面  认为是数学运算



1. 对象属性名： 可以是基本数据类型值；可以支持Map数据结构作为属性名

2. 对象属性名不能是对象  所以遇到对象的时候变为字符串  toString

3. ```js
   var b = {n:'1'};
   var a = {}
   a[b] = 2
   //此时a ==> {[object Object]:2}
   ```

4. a = a.x = {n:1}

5. ```js
   var a = {n:1}
   var b = a;
   a.x = a = {n:2}  //此时先让对象赋值给a.x 再让对象赋值给 a
   // 先让   a.x  = 这个值的地址
   // 再 让  a = 这个地址
   ```

6. 把函数体中的代码当做字符串存储到堆中 “代码字符串”  =>创建函数不执行，函数没啥用

7. 函数也是对象，他也有自己的键值对   

8. 定义了所在作用域 = 当前创建函数的上下文





### 数组方法整理

- isArray() 判断是否是数组
- from()  类数组转为数组
- Array.of()  和数组构造器一样  但是规避了副作用
- concat()  连接2个或者多个数组
- copyWithin 浅复制
- entries   返回数组的迭代器对象
- every 遍历所有元素看是否满足一个回调函数
- fill 
- filter
- find
- findIndex 
- flat 数组扁平化  [[12]].flat(infinity)
- flatMap  类似map
- forEach
- Includes 
- indexOf 
- join
- Keys 返回索引的数组
- lastIndexOf
- map
- pop
- push
- reduce
- reduceRight
- reverse
- shift
- unshift
- Slice
- some 
- Sort
- splice
- toString
- Unshift
- Values()



parseInt ：parseInt([value],[radix]) 多支持一个进制基数

parseFloat : 多识别一个小数点

0x是16进制标识



### VO  AO  

- GO 全局对象window    堆内存   浏览器内置的API
- VO(G) 全局变量对象   上下文中的空间   全局上下文中创建的变量
- 基于VAR/FUNCTION在全局上下文中声明的全局变量也会给GO赋值一份（映射机制）
- 但是就LET/CONST等ES6方式在全局上下文中创建的全局变量和GO没有关系



### 面试公开课

- js

  - 数据类型和堆栈内存
  - 垃圾回收机制
  - ES6核心
  - 同步异步
  - 闭包
  - 面向对象

- ajax和HTTP

  - HTTP网络相关
  - 性能优化
  - 跨域

- webpack

  - 原理

- 框架

  - dom diff
  - 原理

- forEach原理

- ```js
  Object.prototype.each = funciton each(){
  //	this  指向调用这个方法的实例
    
  }
  ```

#### This 指向问题

- this的指向和执行环境无关，只和调用者有关例如

- ```js
  function fn(){
    console.log(this)
  }
  document.body.onclick = function(){
    // this 指向dom
    fn() // 指向window
  }
  ```

  

### 函数柯里化

- 预先存储或者预先处理的概念
- 就是外层函数执行返回一个函数  基于作用域链机制找到闭包中存储的信息拿来使用；形成的闭包类似于预先存储；
- redux 源码中就是利用柯里化思想， 大函数执行，返回一个小函数
- redux , vuex 源码都是应用柯里化思想

### compose函数

- 函数组合

  ```js
  const add1 = (x) => x+2;
  const mul3 = (y) => y+2;
  const div2 = (z) => z+2;
  div2(mul3(add1(add1(0))))
  
  compose(x,y,z)  //  实现 div2(mul(add(1)))
  const operate = compose(div2,mul3,add1,add1)
  operate(0) // 相当于 div2(mul3(add1(add1(0))))
  // 一个函数的执行结果当做实参给另外一个函数
   // 想使用一个函数讲这些函数串起来， 这个函数接受任意多个函数作为参数，这些函数都值接受一个参数
  const operate = compose(div2,mul3,add1,add1)
  function compose(...funcs){
    // funcs 就是所有传递进来的函数
    return anonymous(val){  //val是第一个函数执行的实参
      // 如果不传递函数进来
      if(funcs.length === 0) return val
      if(funcs.length === 1) return funcs[0](val)
      /*funcs.reverse().reduce((N,item) => {
        typeof N === 'function' ? item(N(val)) : item(N)
      })*/
      funcs.reverse().reduce((N,item) => item(N),val)
    }
  }
  compose(div2,mul3,add1,add1)(0)
  ```

  ```js
  arr = [10,20,30,40]
  
  arr.reduce((N, item) =>{
    // item是每一次迭代的值
    // N是上一次函数处理返回的结果
    // 例如 我函数中返回 N+item 则第二次 N的值就是10
    return N + item // 作为每次迭代N的值
  },0) // 0是第一次的值, 第一次赋值给N = 0 ，如果不传，N是数组第一项，遍历从从第二项开始
  ```

  

# 面向对象



- ### new Fn和new Fn()

  - 都是创建类的实例
  - new Fn ()可以传参,没有其他区别

#### 函数数据类型和对象数据类型

- 函数数据类型： 普通函数和类
- 对象数据类型： 普通对象、数组对象，正则对象、日期对象，实例对象，函数对象（和普通对象一样，有自己的键值对）

### 原型原型链

- 属性检测

  - Instanceof： 对象是否是类的实例 
  - hasOwnproperty ：实例是否有自己的私有属性 
  - in 某个属性是否在是对象的属性

- 规律

  - 每一个**类（函数）**都具备**prototype属性**，并且属性**是一个对象**（除开箭头函数，箭头函数没有prototype属性），**对象中会存储公共属性和方法**

- 原型对象天生具备一个属性constructor，在prototype的堆内存中，是浏览器则浏览器会默认开辟一个堆内存，会存在这个属性constructor，指向类本身（Fn.prototype.constructor = Fn）

  - 每一个对象都具备内置属性：\__proto__，属性值是当前实例所属类的原型

  

  

  ```js
  function Fn(){
  	this.x = x;
    this.y = y;
    this.getX = function(){
      console.log(this.x)
    }
  }
  let f1 = New Fn()
  Fn.prototype.constructor = Fn
  
  
  
  // 重要方法
  assign 
  create
  defineProperty
  entries 获取属性值
  getPrototypeOf
  is 做对比
  keys
  
  obj.hasOwnProperty('attr') // 检测某个属性是否是对象的**私有属性**
  obj是基于原型链查找 找到Object.prototype.hasOwnProperty，并且把它执行的；
  
  ```

   

  所有的类都是函数数据类型，是一个堆

  所有类的原型也是一个对象  prototype == Object

  Object.prototype 也是一个普通对象 所以也是Object的实例，对象的基类上的

  

  **明确某个方式或者属性是相对于公有还是私有**

  

  

  ```js
  function Fn() {
      this.x = 100;
      this.y = 200;
      this.getX = function () {
          console.log(this.x);
      }
  }
  Fn.prototype.getX = function () {
      console.log(this.x);
  };
  Fn.prototype.getY = function () {
      console.log(this.y);
  };
  let f1 = new Fn;
  let f2 = new Fn;
  console.log(f1.getX === f2.getX);  //  先找自己私有的  自己函数堆中的私有方法都有getX 所以不会相等
  console.log(f1.getY === f2.getY);  // 自身私有堆中没有getY这个方法，所以去Fn.prototype中去找，公用了getY这个方法， 所以是相等的
  console.log(f1.__proto__.getY === Fn.prototype.getY); // f1.__proto__是跳过查找自己私有的，直接基于原型链找所属类原型上的(IE禁止使用__proto__)，getY都是Fn.prototype 对象中的方法，所以是相等的
  console.log(f1.__proto__.getX === f2.getX);  // f1.__proto__.getX是去找公有的，f2是私有堆中有的，所以不相等
  console.log(f1.getX === Fn.prototype.getX); // f1私有堆中有getX 所以不相等
  console.log(f1.constructor);  // 指向构造函数 Fn
  console.log(Fn.prototype.__proto__.constructor);  // Object
  f1.getX();  // 100
  f1.__proto__.getX();  // undefined
  f2.getY(); // 200 
  Fn.prototype.getY();// undefined
  
  ```

  ```js
  function fun(){
      this.a=0;
      this.b=function(){
          alert(this.a);
      }
  }
  // 函数原有的原型对象重定向到这个对象了 之前的原型对象会被回收 会丢失constructor属性 
  // 一般会手动设定constructor
  fun.prototype={
      b:function(){
          this.a=20;
          alert(this.a);
      },
      c:function(){
          this.a=30;
          alert(this.a)
      }
  }
  var my_fun=new fun();
  my_fun.b();
  my_fun.c();
  
  
  ```

  实例.方法()  

  - 方法执行的时候  方法中的this就是当前处理的实例

    - 扩展的方法名字最好设置前缀
    - this的结果一定是对象数据类型值、向基本数据类型的原型上扩展方法，方法执行的时候，方法中的this不再是基本数据类型；但是还是按照原始方法处理
    - 如果返回的结果依然是类的实例，还可以继续调用原型上其他方法

  - eg

  - ```js
    let res = 12.minus(5).add(7)
    // 需要对num参数进行处理
    function handleNum(num){
      num = Number(num)
      return  isNaN(num)?0:num
    }
    Number.prototype.minus = function(num){
      return this - num
    }
    Number.prototype.add - function(num){
      return this + num
    }
    console.log(res === 14) 
    ```

  - 

  

  

  ### 原型链查找机制：

  - 当调用当前实例对象的某个属性（成员访问），先查看自己私有属性是否存在，存在则调用自己私有的属性，不存在， 则默认通过\__proto\__查找所属父类prototype上的公有属性和方法；如果还没有，再基于prototype上的 \___proto\__  继续想上级查找，知道找到Object.prototype 为止
  - 

  

  new 的实现

  ```js
  // 让一个类的__proto__指向 所属类的原型  才是创建这个类的实例对象,
  // 实例的原型链__proto__一定指向他的原型prototype
  实现new 达到这样的效果
  let sanmao = _new(Dog,'三毛')
  sanmao.bark()//
  sanmao.sayName()//
  console.log(sanmao instanceof Dog)
  function _new(Func,...args){
    // 1,创建一个实例对象(让对象.__proto__ === 类.prototype)，这样才创建了Func的实例对象
    let obj = {}
    obj.__proto__ = Func.prototype;
    // IE中禁止使用__proto__
    //  所以 使用如下代替上面2步 
    // Object.create(xx) 会创建一个空对象 ,同时把xx作为当前对象的原型链指向
    // 正常创建一个对象obj  其__proto__指向的是Object.prototype ，现在让他指向我们传递进来的函数的原型
    let obj = Object.create(Func.prototype)
    
    // 2,把类当做普通函数执行(this指向实例对象)
    let res = Func.call(obj,...args)
    
    // 3，看一下函数执行是否存在返回值，不存在或者返回的是值类型，则默认返回实例，如果返回的是引用类型则返回的不是实例而却自己写的引用类型值
    // new操作符：简单来说就是以传入进来函数的返回结果为主，new一定是返回引用类型值，如果是值类型我们就返回我们自己创建的，否则返回函数中显式返回的对象为主
    if(result !== null && /^(object|function)$/.test(typeof res)){
      return result
    }
    // 以自己返回的结果为主，否则返回obj
  }
  ```

  #### 重写Object.create

  ```js
  // 创建某个类的空实例
  Object.create = function create(prototype){
    function Fun(){}
    Fun.prototype = prototype;
    return new Fun();
  }
  
  
  ```

  ```js
  let obj = {
    2: 3,
    3: 4,
    length: 2,
    push: Array.prototype.push
  }
  obj.push(1)
  obj.push(2);
  console.log(obj)
  // 搞清楚内置push 做的事如下
  Array.prototype.push = function(val){
    //1 向数组 (this)末尾追加新的内容，数组索引连续的，所以增加的这一项的索引肯定在最大索引上加1
    this[this.length] = val
    // 2 原始数组this 的长度在之前的基础上自动+1
    this.length ++ ;
    //3 返回新增后数组的长度
  }
  // 按照push的所做的事情分析
  ```

  

  ```js
  // var a = ?
  if(a == 1 && a == 2 && a == 3){
    console.log('ok')
  }
  
  // == 比较，如果左右左右两边数据类型不一样  对象 == 字符串  把对象转为字符串，剩下的情况都要转数字
  // 基本数据类型转数字，默认隐式调用Number()来处理
  // 对象转数字；先转为字符串（先调用valueOf ，获取原始值，如果原始值不是基本类型，继续调用toString ）,然后把字符串转数字
  
  var a = {
    i:1,
    valueOf(){
      return this.i++
    }
  }
  ```

  

  ```js
  function Foo() {
      getName = function () {
          console.log(1);
      };
      return this;
  }
  Foo.getName = function () {
      console.log(2);
  };
  Foo.prototype.getName = function () {
      console.log(3);
  };
  var getName = function () {
      console.log(4);
  };
  function getName() {
      console.log(5);
  }
  Foo.getName();  // 2
  getName(); // 4
  Foo().getName(); // 1
  getName();  // 4
  new Foo.getName();  // 
  new Foo().getName();
  new new Foo().getName();
  ```

  ### this的五中情况

  - 事件绑定
  - 普通函数执行
  - 构造函数执行
  - 箭头函数
  - call/apply/bind

  ```js
  /*
  全局上下文中的this是window ;
  块级上下文中没有自己的this继承所在上下文的this;
  this不是函数执行上下文，this是执行主体
  主要考虑函数私有上下文中，this的情况
  */
  ```

  - 事件执行主体：**函数中的this和在哪执行没关系** 

    1. 事件绑定，给元素的某个事件绑定方法；当事件行为处罚，方法执行，方法中this是当前元素本身
    2. 普通方法执行（包含自执行，普通函数，对象成员访问调取方法等），只需要看函数执行主体，就是方法.前面是谁 this就是谁

    ```js
    函数中的this和在哪执行没关系
    (function(){
      console.log(this) // window
    })()
    
    let obj = {
      fn: (function(){
        console.log(this) // window
      })()
    }
    ```

    3. 构造函数中的this 如果是用构造函数模式执行则指向这个实例，[].slice() ： 找到Array原型上的slice方法，然后再把slice方法执行，slice方法中的this是当前空数组，原型上的方法不一定都是这个实例，主要看是谁调用

        

       ```js
       function Func() {
         console.log(this)  // obj = new Func  指向这个实例obj
       }
       Func.prototype.getNum = function getNum(){
         console.log(this)  // 看执行主体
       }
       ```

       

    4. 箭头函数this继承上下文中的this

    ```js
    let obj = {
      func: function(){
        console.log(this)
      },
      sum: () => {
        console.log(this)
      }
    }
    
    obj.func()  // this -> obj
    obj.sum() // this指向上下文： window
    obj.sum.call(obj) // 箭头函数没有this 所以this还是window
    
    不建议乱用箭头函数
    ```

    ```js
    let obj = {
      i: 0,
      func(){
        setTimeout(function(){
          this.i++
        }.bind(this),1000)  // 通过bind将this绑定为外层的
      }
    }
    ```

    5. call/apply/bind

  ```js
  var num = 10;
  var obj = {
    num:20
  }
  obj.fn = (function(num){
    this.num = num *3; // window num 60
    num++;  // obj.num 21
    return function(n){
      this.num += n; //  window // 35
      num++;
      console.log(num)
    }
  })(obj.num)
  var fn = obj.fn;
  fn(5);
  obj.fn(10);
  console.log(num,obj.num)  //
  
   // 将函数执行的结果赋值给 obj.fn
  
  ```

- ```js
  Array.of 
  Array.from // 类数组转数组
  Array.isArray 检测数组
  ```



- 每一个函数(包含内置类)都是Function的实例；所以其\__proto\___一定指向Function.prototype

- 每一个对象（包含原型对象）都是Object的实例，所以最后其\__proto\___一定可以指向Object.prototype

- Object也可以作为构造函数  所以其\__proto\__ 也指向Function.prototype

- Function instanceof Function  true

- Function.prototype === Function.\__proto__  type

  ```js
  // Object做为一个类（一个函数）它是Function的一个实例；Function虽然是函数（类）但他也是一个对象，所以它也是Object的一个实例；
  // Object.__proto__.__proto__===Object.prototype  TRUE
  -----------------------
  // 在JS中的任何实例（任何值【除了值类型的值】）最后都可以基于自己的__proto__找到Object.prototype，也就是所有的值都是Object的实例 =>万物皆对象
  ```

  

- 非严格模式下  call apply 中传递null undefined 不传 this指向window
- 严格模式下传递谁就是谁

```js
bind原理就是外层一个函数改变里面的函数
let body = document.body;
let obj = {
  name: 'obj'
}
function func (x,y){
  console.log(this,x,y)
}
// 我现在需要点击的时候让func里面的this指向obj
body.onclick = func.bind(obj,10,20)  // 
// 等同于
// 把返回的匿名函数赋值给了事件绑定
body.onclick = function anonymous(){
  func.call(obj,10,20)
}
// bind原理执行bind方法，返回匿名函数

```





### 数据类型检测

- typeof 
  - 检测简单，语法简单
  - 缺点：检测数组，null, 对象，正则 都是'object',无法区分对象数据类型
- instanceof
  - 基于原型检测；检测这个值是否属于这个类，检测是否是这类型，数组，正则，对象可以细分一下
  - 但是基本数据类型无法检测，检测原理：只要在当前实例的\__proto\__出现这个类，检测结果都是true
- constructor
  - 和instanceof类似，也是非专业检测数据类型；val.constructor === 类
  - 因为获取实例的constructor实际上获取的是直接所属的类，所以在检测准确性上比instanceof好点，但是constructor是可以修改的,所以也是不准确的
- Object.prototype.toString.call(val) 可以简写({}).toString.call
  - 



  

  

  

  

  

 























