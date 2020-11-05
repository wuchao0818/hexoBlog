---
title: ES6 新特性
date: 2019-06-10 16:33:08
categories: web前端
tags:
- javascript
---

### 1、变量生命：const 和 let

ES6推荐使用 let 声明局部变量，相比之前的 var（无论声明在何处，都会被视为声明在函数的最顶部）
let 和 va r声明的区别：

```javascript
var x = '全局变量';
{
  let x = '局部变量';
  console.log(x); // 局部变量
}
console.log(x); // 全局变量
```

let 表示声明变量，而 const 表示声明常量，两者都为块级作用域；const  声明的变量都会被认为是常量，意思就是它的值被设置完成后就不能再修改了

```javascript
const a = 1
a = 0 //报错
```

有几个点需要注意：

> - let 关键词声明的变量不具备变量提升（hoisting）特性
> - let 和 const 声明只在最靠近的一个块中（花括号内）有效
> - 当使用常量 const 声明时，请使用大写变量，如：CAPITAL_CASING
> - const 在声明时必须被赋值

#### 变量提升

```javascript
console.log(a);//undefined

var a = "hey I am now hoisting";
```

看起来，我们在 a 被声明前调用 a，没有报错，反而是返回一个 undefined 值，原因是：a 其实已经在调用前被声明了，只是没有被初始化。JavaScript 会把作用域里的所有变量和函数提到函数的顶部声明，相当于：

```javascript
var a;
console.log(a);//undefined

a = "hey I am now hoisting";
```

由此可以知道，变量提升指的是变量声明的提升，不会提升变量的初始化和赋值。

对于let来说，情况有点不同:

```javascript
console.log(a);//Uncaught ReferenceError: a is not defined

let a = "hey I am now hoisting";
```

### 2、模板字符串

在ES6之前，我们往往这么处理模板字符串：通过 “\” 和 “+” 来构建模板

```javascript
$("body").html("This demonstrates the output of HTML \
content to the page, including student's\
" + name + ", " + seatNumber + ", " + sex + " and so on.");
```

而对ES6来说

1. 基本的字符串格式化。将表达式嵌入字符串中进行拼接。用${}来界定；
2. ES6反引号(``)直接搞定；

```javascript
$("body").html(`This demonstrates the output of HTML content to the page, 
including student's ${name}, ${seatNumber}, ${sex} and so on.`);
```

### 3、箭头函数

ES6 中，箭头函数就是函数的一种简写形式，使用括号包裹参数，跟随一个 =>，紧接着是函数体；

```jsx
// ES5
var add = function (a, b) {
    return a + b;
};
// 使用箭头函数
var add = (a, b) => a + b;

// ES5
[1,2,3].map((function(x){
    return x + 1;
}).bind(this));
    
// 使用箭头函数
[1,2,3].map(x => x + 1);
```

#### 区别：

- 箭头函数是匿名函数，不能作为构造函数，不能使用new
- 箭头函数不绑定this，会捕获其所在的上下文的this值，作为自己的this值
- 箭头函数没有原型属性
- 箭头函数不能换行

#### this 指向问题

普通函数：根据调用我的人（谁调用我，我的this就指向谁）

箭头函数：根据所在的环境（我再哪个环境中，this就指向谁）

```javascript
  function fun(){
        console.log(this);          
    }
    fun() //直接调用,this指向window
    btn.onclick = function () {
    //给btn这个元素绑定点击事件，一点击就会执行fun函数，也就是相当于btn来执行fun函数，
    //那么fun中this指向这个btn这个对象
        console.log(this); //指向btn

    }
```

```javascript
    var x = 10;
    var obj = {
       x:20,
       say:function(){
           console.log(this.x);

       }
    }
    obj.say()//输出20
```

obj 调用了 say 方法的函数，所以 this 的指向是 obj,所以在这里输出的值为 20，正是应合了普通函数中的这句话：**谁调用这个函数，那么函数的thi就指向谁**



#### 箭头函数中没有所谓的this指向

##### 这个this跟箭头函数在哪里调用没有关系，但是跟箭头函数定义在哪里有关系

```javascript
 let obj = {
      name:"aa",
      say:()=>{
          console.log(this);//定义所处的位置实在window这个位置的，所以这里的this指向                                 window        
      }
  }
  console.log(obj.say());
```

然后我们在定义一个普通函数在obj对象里面

```javascript
let obj = {
          name:"aa",
          say:()=>{
              console.log(this);//定义所处的位置实在window这个位置的，所以这里的this指向window        
          },
          go:function(){
              console.log(this);//这个this指向obj.因为这个go方法是由obj来调用的，所以this指向obj
              
          }
      }
      console.log(obj.say());
      console.log(obj.go());
```

```javascript
let obj = {
            name: "aa",
            go: function () {
                console.log(this);
                let say = () => {
                    console.log(this);//此时这个say的this指向obj这个对象 
                }
                say();

            }
        }
        console.log(obj.go());
```

say 在 go 方法的函数里面，此时的 go 是指向 obj 的，所以 say 会跟着指向 obj



### 4、对象属性简写

在ES6中允许我们在设置一个对象的属性的时候不指定属性名

> 不使用ES6

```javascript
const name='Ming',age='18',city='Shanghai';
   
const student = {
    name:name,
    age:age,
    city:city
};
console.log(student);//{name: "Ming", age: "18", city: "Shanghai"}
```

> 使用ES6

```javascript
const name='Ming',age='18',city='Shanghai';
  
const student = {
    name,
    age,
    city
};
console.log(student);//{name: "Ming", age: "18", city: "Shanghai"}
```

对象中直接写变量，非常简洁。

### 5、模块化

ES5不支持原生的模块化，在ES6中模块作为重要的组成部分被添加进来。模块的功能主要由 export 和 import 组成。每一个模块都有自己单独的作用域，模块之间的相互调用关系是通过 export 来规定模块对外暴露的接口，通过import来引用其它模块提供的接口。同时还为模块创造了命名空间，防止函数的命名冲突。

**`导出(export)`**
ES6允许在一个模块中使用export来导出多个变量或函数。

导出变量

```javascript
//test.js
export var name = 'Rainbow'


 var name = 'Rainbow';
 var age = '24';
 export {name, age};
```

导出函数

```javascript
// myModule.js
export function myModule(someArg) {
  return someArg;
} 
```

**`导入(import)`**
定义好模块的输出以后就可以在另外一个模块通过 import 引用。

```javascript
import {myModule} from 'myModule';// main.js
import {name,age} from 'test';// test.js
```

在不使用`{}`来引用模块的情况下，`import`模块时的命名是随意的

```javascript
// A.js
export default 42
```

```javascript
// B.js
import A from './A'
```

### 6、promise

> Promise对象用于异步操作，它表示一个尚未完成且预计在未来完成的异步操作。

我们知道，JavaScript的执行环境是「单线程」。 
所谓单线程，是指JS引擎中负责解释和执行JavaScript代码的线程只有一个，也就是一次只能完成一项任务，这个任务执行完后才能执行下一个，它会「阻塞」其他任务。这个任务可称为主线程。 
但实际上还有其他线程，如事件触发线程、ajax请求线程等。

这也就引发了同步和异步的问题。

**同步模式**，即上述所说的单线程模式，**一次**只能执行**一个**任务，函数调用后需等到函数执行结束，返回执行的结果，才能进行下一个任务。如果这个任务执行的时间较长，就会导致「**线程阻塞**」

**异步模式**，即与同步模式相反，可以一起执行**多个任务**，函数调用后不会立即返回执行的结果，如果任务A需要等待，可先执行任务B，等到任务A结果返回后再继续回调。

#### 基本用法

`Promise`对象代表一个未完成、但预计将来会完成的操作。
它有以下三种状态：

- `pending`：初始值，不是fulfilled，也不是rejected
- `fulfilled`：代表操作成功
- `rejected`：代表操作失败

```javascript
 /* 例3.1 */
 2 //构建Promise
 3 var promise = new Promise(function (resolve, reject) {
 4     if (/* 异步操作成功 */) {
 5         resolve(data);
 6     } else {
 7         /* 异步操作失败 */
 8         reject(error);
 9     }
10 });
```

类似构建对象，我们使用 `new` 来构建一个`Promise`。`Promise`接受一个「函数」作为参数，该函数的两个参数分别是`resolve`和`reject`。这两个函数就是就是「回调函数」，由JavaScript引擎提供。

Promise实例生成以后，可以用`then`方法指定`resolved`状态和`reject`状态的回调函数。

```javascript
2 promise.then(onFulfilled, onRejected);
3 
4 promise.then(function(data) {
5   // do something when success
6 }, function(error) {
7   // do something when failure
8 });
```

Promise新建后就会立即执行。而`then`方法中指定的回调函数，将**在当前脚本所有同步任务执行完才会执行**。如下例：

```javascript
setTimeout( function(){
    console.log(1)
},0);
new Promise(function(resolve){
    console.log(2)
    for( var i=0 ; i<10000 ; i++ ){
        i==9999 && resolve()
    }
    console.log(3)
}).then(function(){
    console.log(4)
});
console.log(5);

 //2 3 5 4 1
```

> - Javascript是单线程的，所有的同步任务都会在主线程中执行。
> - 当主线程中的任务，都执行完之后，系统会 “依次” 读取任务队列里的事件。与之相对应的异步任务进入主线程，开始执行。
> - 异步任务之间，会存在差异，所以它们执行的优先级也会有区别。大致分为 微任务（micro task，如：Promise、MutaionObserver等）和宏任务（macro task，如：setTimeout、setInterval、I/O等）。
> - Promise 执行器中的代码会被同步调用，但是回调是基于微任务的。
> - 微任务的优先级高于宏任务
> - 每一个宏任务执行完毕都必须将当前的微任务队列清空