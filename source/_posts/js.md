# js

### 1.执行上下文

#### 分类

**全局执行上下文** — 这是默认或者说基础的上下文，任何不在函数内部的代码都在全局上下文中。它会执行两件事：创建一个全局的 window 对象（浏览器的情况下），并且设置 `this` 的值等于这个全局对象。一个程序中只会有一个全局执行上下文。

**函数执行上下文** — 每当一个函数被调用时, 都会为该函数创建一个新的上下文。每个函数都有它自己的执行上下文，不过是在函数被调用时创建的。函数上下文可以有任意多个。每当一个新的执行上下文被创建，它会按定义的顺序（将在后文讨论）执行一系列步骤。

**Eval 函数执行上下文** — 执行在 `eval` 函数内部的代码也会有它属于自己的执行上下文，但由于 JavaScript 开发者并不经常使用 `eval`，所以在这里我不会讨论它。

#### 组成

**ES6：**

```javascript
ExecutionContext = {
  ThisBinding = <this value>,  // this 值的决定，即我们所熟知的 This 绑定。
  LexicalEnvironment = { ... }, // 词法环境组件。
  VariableEnvironment = { ... },// 变量环境组件。
}
```

词法环境 ：

现在，在词法环境的**内部**有两个组件：

1. **环境记录器**是存储变量和函数声明的实际位置。
2. **外部环境的引用**意味着它可以访问其父级词法环境（作用域）。

变量环境：

在 ES6 中，**词法环境**组件和**变量环境**的一个不同就是前者被用来存储函数声明和变量（`let` 和 `const`）绑定，而后者只用来存储 `var` 变量绑定。



**ES3规范：**

对于每个执行上下文，都有三个重要属性：

- 变量对象(Variable object，VO)
- 作用域链(Scope chain)
- this

变量对象：

- 函数的所有形参 (如果是函数上下文)
- 函数声明
- 变量声明

作用域链：

每个函数创建的时候，会有一个内部的属性[[scope]],就会保存**所有父变量对象**到其中。

当函数激活时，进入函数上下文，创建 VO/AO 后，就会将活动对象添加到作用链的前端，这时候执行上下文的作用域链，我们命名为 **Scope**：这才叫做作用域链

```javascript
Scope = [AO].concat([[Scope]]);
```



### 2.执行栈

执行栈，也就是在其它编程语言中所说的“调用栈”，是一种拥有 LIFO（后进先出）数据结构的栈，被用来存储代码运行时创建的所有执行上下文。

### 3.静态作用域和动态作用域

**js采用的是词法作用域，也就是静态作用域。**

```javascript
var value = 1;

function foo() { 
    console.log(value);
}

function bar() {
    var value = 2;
    foo();
}

bar();
```

假设JavaScript采用静态作用域，让我们分析下执行过程：

执行 foo 函数，先从 foo 函数内部查找是否有局部变量 value，**如果没有，就根据书写的位置，查找上面一层的代码**，也就是 value 等于 1，所以结果会打印 1。

假设JavaScript采用动态作用域，让我们分析下执行过程：

执行 foo 函数，依然是从 foo 函数内部查找是否有局部变量 value。**如果没有，就从调用函数的作用域**，也就是 bar 函数内部查找 value 变量，所以结果会打印 2。

前面我们已经说了，**JavaScript采用的是静态作用域**，所以这个例子的结果是 1。

### 4.this

在JavaScript中，`this`的指向是调用时决定的，而不是创建时决定的，这就会导致`this`的指向会让人迷惑，简单来说，`this`具有**运行期绑定**的特性。

在一个函数上下文中，this由调用者提供，由调用函数的方式来决定。**如果调用者函数，被某一个对象所拥有，那么该函数在调用时，内部的this指向该对象。如果函数独立调用，那么该函数内部的this，则指向undefined**。但是在非严格模式中，当this指向undefined时，它会被自动指向全局对象。

```javascript
'use strict';
var a = 20;
function foo() {
  var a = 1;
  var obj = {
    a: 10,
    c: this.a + 20,
    fn: function () {
      return this.a;
    }
  }
  return obj.c;

console.log(foo());    //  Uncaught TypeError: Cannot read properties of undefined (reading 'a')
console.log(window.foo());  //   40
```

```javascript
function foo() {
  console.log(this.a)
}

function active(fn) {
  fn();
}

var a = 20;
var obj = {
  a: 10,
  getA: foo
}

active(obj.getA);  // 20
```

### 5.创建二维数组的两种方式

new Array(n).fill().map(()=>{return new Array(n).fill(false)})

Array(m).fill().map(() => Array(n))

Array.from(Array(m), () => new Array(n))

Array.from(new Array(3),()=>{return new Array(3).fill(false)})

### js类型判断

- **typeof**：能判断所有**值类型，函数**。不可对 **null、对象、数组**进行精确判断，因为都返回 `object`
- **instanceof**：能判断**对象**类型，不能判断基本数据类型，**其内部运行机制是判断在其原型链中能否找到该类型的原型**
- **Object.prototype.toString.call()**：所有原始数据类型都是能判断的，还有 **Error 对象，Date 对象**等。

### 判断是否位数组

```javascript
Array.isArray(arr); // true
arr.__proto__ === Array.prototype; // true
arr instanceof Array; // true
Object.prototype.toString.call(arr); // "[object Array]"
```

### 深拷贝

```javascript
// 考虑数组，考虑循环引用（用weakmap代替map）
function deepClone(target,map = new WeakMap()){
   	if(typeof target == "object"){
        let cloneTarget = Array.isArray(target) ? [] : {};
        if(map.has(target)){
            return map.get(target);
        }
        map.set(target,cloneTarget);
        for(let key in target){
            cloneTarget = deepClone(target);
        }
        return cloneTarget;
    }else{
        return target;
    }
}
```

### 浏览器回收机制

- **标记清除**：标记阶段即为所有活动对象做上标记，清除阶段则把没有标记（也就是非活动对象）销毁。
- **引用计数**：它把**对象是否不再需要**简化定义为**对象有没有其他对象引用到它**。如果没有引用指向该对象（引用计数为 0），对象将被垃圾回收机制回收。

标记清除的缺点：

- **内存碎片化**，空闲内存块是不连续的，容易出现很多空闲内存块，还可能会出现分配所需内存过大的对象时找不到合适的块。
- **分配速度慢**，因为即便是使用 First-fit 策略，其操作仍是一个 O(n) 的操作，最坏情况是每次都要遍历到最后，同时因为碎片化，大对象的分配效率会更慢。

引用计数的缺点：

- 需要一个计数器，所占内存空间大，因为我们也不知道被引用数量的上限。
- 解决不了循环引用导致的无法回收问题。

V8 的垃圾回收机制也是基于标记清除算法，不过对其做了一些优化。

- 针对新生区采用并行回收。
- 针对老生区采用增量标记与惰性回收。

### 防抖

```javascript
function debouce(func,wait,immediate){
	let timeout;
	return function fn(){
		let context = this;
        let args = arguments;
        
        if(timeout) clearTimeout(timeout);
        if(immediate){
            let callNow = !timeout;
            timeout = setTimeout(function(){
                timeout = null;
            },wait)
            if(callnow) func.apply(context,args);
        }else{
            timeout = setTimeout(function(){
                func.apply(context,args);
            },wait);
        }
	}
}
```

### 节流

```javascript
function throttle(func,wait){
    let timeout;
    
    return funcion fn(){
        let context = this;
        let args = arguments;
        
        if(!timeout){
            timeout = setTimeout(function(){
                timeout = null;
                func.apply(context,args);
            },wait);
        }
        
    }
}
```

## es6

### promise

方法：

- [Promise.prototype.then()](https://es6.ruanyifeng.com/#docs/promise#Promise.prototype.then())
- [Promise.prototype.catch()](https://es6.ruanyifeng.com/#docs/promise#Promise.prototype.catch())
- [Promise.prototype.finally()](https://es6.ruanyifeng.com/#docs/promise#Promise.prototype.finally())
- [Promise.all()](https://es6.ruanyifeng.com/#docs/promise#Promise.all())
- [Promise.race()](https://es6.ruanyifeng.com/#docs/promise#Promise.race())
- [Promise.allSettled()](https://es6.ruanyifeng.com/#docs/promise#Promise.allSettled())
- [Promise.any()](https://es6.ruanyifeng.com/#docs/promise#Promise.any())
- [Promise.resolve()](https://es6.ruanyifeng.com/#docs/promise#Promise.resolve())
- [Promise.reject()](https://es6.ruanyifeng.com/#docs/promise#Promise.reject())
- [Promise.try()](https://es6.ruanyifeng.com/#docs/promise#Promise.try()) : 同步的代码同步执行，异步代码异步执行；捕获同步代码的错误；

