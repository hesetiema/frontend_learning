# Javascript 进阶🌞

## 一、数据类型

### 1、基本类型

基本类型（原始类型）数据**值存储在栈(stack)中**。基本包装类型`Number、String、Boolean`作为内置对象，是一种特殊的引用类型。

- number：采用 IEEE754 标准中**双精度浮点数 64 位**(8 个字节)存储，不区分整数

  - 符号位(1 bit)
  - 指数位(11 bit)：$0 \leq e \leq 2^{11} -1=2047$
    - 偏移结果：$-1023 \leq e-bias \leq 1024$
    - 数值范围：[Number.MIN_VALUE,Number.MAX_VALUE]=[$2^{-1023}$,$2^{1024}$]
  - 小数位(52 bit)

    - 数值精度：IEEE754 规定，有效数字第一位默认总是 1 。因此，在表示精度的位数前面，还存在一个 “隐藏位” ，固定为 1
    - 可安全精确运算的整数范围：[$-2^{53}-1$，$2^{53}-1$]
      - 检查安全性：使用 ES6 `Number.isSafeInteger()`
      - 大整数计算：针对超出安全整数范围的大整数，在 es10 中加入了第七种原始类型`BigInt`，可操作超过最大安全数字的数字

    ```js
    console.log({
      max: Number.MAX_SAFE_INTEGER,
      min: Number.MIN_SAFE_INTEGER
    });
    // {max: 9007199254740991, min: -9007199254740991}
    ```

    - **精度丢失**：**浮点数转化为二进制出现无限循环时，必须 0 舍 1 入处理**
      - 如`console.log(.1 + .2); // 0.30000000000000004`
      - 如果要使用任意精度的十进制运算，请使用`big.js`，此库不适用于 UI /性能密集型目的的数学运算，例如图表，画布绘制等

- string：采用 UTF-16（16 位 Unicode 字符）编码 ，中英文字符均占用 2 个字节
- boolean：两字面值：true、false，所有类型值都可转化为与 boolean 等价的值
- symbol：表示唯一的标识符。可作为对象唯一的属性名，创建对象的“隐藏”属性
- undefined：未定义的值。表示一个变量最原始的状态，而非人为操作的结果
- null：表示一个对象被人为重置为空，而非一个变量最原始的状态
  - typeof null == 'object'：
    - **二进制的开头 000 会被 typeof 判定为对象类型， 而 null 值的二进制表示全是 0。故将其误判为对象类型**
  - 获得正确类型方法：
    - `Object.prototype.toString.call(null);// [object Null]`

### 2、引用类型

引用类型数据**值存储在堆(heap)中，变量指针将内存地址存储到栈中**。引用类型除 Object 本身外，Array、Function、Date、RegExp 也属于引用类型

- 底层数据结构：所有 js 对象都是基于一个名为 HeapObject 的类生成的。生成对象的时候，引擎通过这个类向 Heap 申请空间
  - 对象的储存结构：头部信息、属性块，存的是 k-v 值形式的集合、元素块，按照内存的结构存储的真正数组
- 浅拷贝：仅能拷贝指向基本类型的对象属性。如`Object.assign({},obj);`
- 深拷贝：彻底拷贝一个对象作为副本，两者间的操作相互不受影响。

## 二、执行上下文、执行栈与内存机制

### 1、执行上下文

执行上下文(EC)是当前 JavaScript 代码被解析和执行时所在环境的抽象概念。

- 类型
  - 全局执行上下文：只有一个，浏览器中的全局对象就是 window 对象
  - 函数执行上下文：存在无数个，每次调用函数都会创建一个新的执行上下文
- 创建
  - **This Binding**：确定 this 的值
    - 全局执行上下文：浏览器中 this->window；Nodejs 中 this->module
    - 函数执行上下文中，this 的值取决于函数的调用方式
  - **LE(词法环境)** 组件创建：**用于存储函数声明和变量(let、const)绑定**
    - 环境记录：存储变量和函数声明的实际位置。如用户在函数中定义的变量被存储在环境记录中，包含了 arguments 对象
    - 对外部环境的引用：可以访问其外部词法环境
  - **VE(变量环境)** 组件创建：**仅用于存储变量(var)绑定**
- 执行：完成对所有变量的分配，最后执行代码

### 2、执行上下文栈

JS 引擎创建了执行上下文栈（Execution context stack，ECS）来管理执行上下文。在函数上下文中，用活动对象(activation object, AO)来表示变量对象。

![avatar](http://resource.muyiy.cn/image/2019-07-24-060256.jpg)

- 函数上下文：调用函数时，自动初始化局部变量 arguments 指代 Arguments 对象。所有作为参数传入的值都会成为 Arguments 对象的数组元素
  - **变量对象(VO)**：包含变量/函数声明、函数的形参，只能在全局上下文中访问
  - **活动对象(AO)**：进入到一个执行上下文后，变量对象被激活为活动对象，各种属性才能被访问
- 执行过程：进入执行上下文、代码执行
- 总结
  - 全局上下文的 VO 初始化是全局对象
  - 函数上下文的 VO 初始化只包括 Arguments 对象
  - 在进入执行上下文时会给 VO 添加形参、函数/变量声明等初始的属性值
  - 在代码执行阶段，会再次修改 VO 的属性值

### 3、内存机制

**自动垃圾收集机制**：收集器每隔一段时间找出不再继续使用的值，释放其占用的内存

- 变量销毁
  - 局部变量：局部作用域中，当函数执行完毕，局部变量无存在的必要
  - 全局变量：很难判断何时需释放内存空间，开发中**尽量避免使用全局变量**
- 垃圾回收算法：标记清除，从根部出发定时扫描内存中的对象，凡是能从根部到达的对象，标记可达，保留；从根部出发无法触及到的对象，未被标记进行回收
- WeakSet/WeakMap：表示这是弱引用，它们对于值的引用都不计入垃圾回收机制

**内存泄漏**：对于不再用到的内存，没有及时释放，就叫做内存泄漏（memory leak）

- 意外的全局变量：未定义的变量或函数调用自身，this 指向全局对象 window
  - 在 JavaScript 文件头部加上 'use strict'，使用严格模式避免意外的全局变量
  - 必须使用全局变量存储大量数据时，确保用完后设置为 null 或重新定义
- 脱离 DOM 的引用：把 DOM 存成字典（JSON 键值对）或者数组，同样的 DOM 元素存在两个引用：DOM 树、字典。那么将来需要把两个引用都清除
- 闭包：匿名函数可以访问父级作用域的变量。可把需清楚的变量设置为 null

## 三、作用域链与闭包

### 1、作用域链

- 含义：当访问一个变量时，解释器会首先在当前作用域查找标示符，没有找到再去父作用域找，直到找到全局对象中或返回 undefined
- 理解：含自身变量对象和上级变量对象的列表，由`[[Scope]]`属性查找上级变量
  如`fContext = {Scope: [AO, checkscopeContext.AO, globalContext.VO],}`

### 2、闭包

闭包，即【**有权访问外层函数作用域中的变量的函数**】。**即使创建变量的上下文已经销毁，变量仍然绑定在内部函数。**

- 优点：它允许您轻松组合对象，例如显示模块模式

## 四、原型与继承

### 1、原型

每个对象拥有一个原型对象，对象以其原型为模板，从原型继承方法和属性，这些属性和方法定义在对象的构造器函数的 prototype 属性上，而非对象实例本身。

- 构造函数：使用 new 生成实例的函数，首字母大写
  - Symbol 不是构造函数，不支持语法 new Symbol()。但其原型上拥有 constructor 属性，即 Symbol.prototype.constructor
- constructor：返回创建实例对象时构造函数的引用
  - 对于引用类型，`constructor`属性值是可以修改的
  - 对于基本类型(null、undefined 无 constructor 属性)，`constructor`属性值只读

```js
function Parent(age) {
  this.age = age;
}
var p = new Parent(50);
p.constructor === Parent; // true
p.constructor === Object; // false
```

![avatar](http://resource.muyiy.cn/image/2019-07-24-060305.jpg)

### 2、原型链

每个对象拥有一个原型对象，通过`__proto__`指针指向上一个原型 ，并从中**继承**方法和属性，同时原型对象也可能拥有原型，这样一层层最终指向 null，这就是原型链。

![avatar](http://resource.muyiy.cn/image/2019-07-24-060308.jpg)

- `__proto__`是每个实例上都有的属性，因为性能问题不推荐使用，推荐使用 `Object.getPrototypeOf()`。但原型链的构建依赖于`__proto__`
- `prototype`是构造函数的属性，在实例上并不存在
- `foo.constructor.prototype==foo.__proto__`

### 3、继承

原型上的方法和属性被**继承**到新对象中，并不是被复制到新对象

- instanceof：判断对象的原型链中是不是能找到类型的 prototype
- 继承方案：原型链继承的本质是重写原型对象，代之以一个新类型的实例。日常使用 ES6 Class extends（模拟原型）继承方案即可

## 五、this 解析、call/apply/bind 原理

### 1、this 解析

- 默认绑定：函数体严格模式下绑定到 undefined，否则绑定到全局对象 window
  - **高阶函数调用最底层时，默认绑定为全局对象 window（箭头函数除外）**
- 隐式绑定：当函数引用有上下文对象时，函数中的 this 绑定到这个上下文对象。
  - 隐式丢失：被隐式绑定的函数特定情况下会丢失绑定对象，应用默认绑定
    - **函数间接引用复制再调用，隐式丢失**
- 显式绑定：通过`func.apply(context,args)、func.call(context,arg1,arg2,...)、func.bind(context,arg1,arg2,...)`方法直接指定 this 的绑定对象
  - 应用场景
    - 创建包裹函数，负责接收参数并返回值
    - 方法借用：`[].join.call(arguments)`借用连接方法
    - 利用 bind()将 this 绑定到调度函数：`setTimeout(func.bind(this),delay)`
- new 绑定：使用 new 来调用构造函数时，this 绑定到新创建的对象，无法被修改

  - **手写一个 new**

```js
//规定 myNew 接收参数的方式：let object1 = myNew(Constructor,arg1,...)
function myNew() {
  // 获得构造函数
  let Con = [].shift.call(arguments);
  // 创建一个空对象链接到原型，obj 可以访问构造函数原型中的属性
  let obj = Object.create(Con.prototype);
  // 绑定 this 实现继承，obj 可以访问到构造函数中的属性
  let result = Con.apply(obj, arguments);
  // 优先返回构造函数返回的对象
  return result instanceof Object ? result : obj;
}
```

- 优先级：**new 绑定->显式绑定->隐式绑定->默认绑定**
- 箭头函数：根据**外层（函数或全局）词法作用域**来决定 this
  - 箭头函数不绑定 this，箭头函数中的 this 相当于普通变量
  - **箭头函数的 this 无法通过`bind，call，apply`来直接修改**（可以间接修改）
  - 改变外层作用域中 this 的指向可以改变箭头函数的 this

### 2、call/apply

- call/apply：`fn.apply(context,args);fn.call(context,...args);//args为参数数组`
  - 借用数组方法
    - `[].join.call(arguments);`类数组对象借用连接方法
  - 借用 Math 的方法
    - `Math.max.apply(Math, numbers);`借用求最大值方法
  - 数据类型检测：`Object.prototype.toString.call(obj)`
- 模拟实现 call

```js
Function.prototype.myCall = function(context) {
  if (typeof this !== "function") {
    throw new TypeError("Error");
  }
  context = context || window; //判断 this 绑定的数据类型
  const fn = Symbol("fn");
  context[fn] = this; //将函数设置为对象的隐形属性
  const args = [...arguments].slice(1);
  const result = context[fn](...args); //执行函数
  delete context[fn];
  return result;
};
```

- 模拟实现 apply

```js
Function.prototype.myApply = function(context, arr) {
  if (typeof this !== "function") {
    throw new TypeError("Error");
  }
  context = context || window; //判断 this 绑定的数据类型
  const fn = Symbol("fn");
  context[fn] = this; //将函数设置为对象的隐形属性
  const result = !arr ? context.fn() : context.fn(...arr);
  delete context[fn];
  return result;
};
```

### 3、bind

- 语法：`fn.bind(thisArg[, arg1[, arg2[, ...]]])`
- 与`call/apply`区别：bind 方法返回一个绑定上下文的函数，而不直接执行函数
- 特点：可以绑定 this、传入参数、柯里化、返回一个函数
- 应用
  - 解决调度里的 this 指向问题（默认指向 window 全局对象）
    - 利用 bind()将 this 绑定到这个函数上：setTimeout(func.bind(this),delay)
    - 利用箭头函数将 this 指向外层调用者：setTimeout(() => {...}, delay)
  - 柯里化：只传递给函数一部分参数，让它返回另一个函数处理剩下的参数
- 模拟实现 bind

```js
Function.prototype.myBind = function(context) {
  if (typeof this !== "function") {
    throw new TypeError("Error");
  }
  const self = this; // this 指向调用者
  const args = [...arguments].slice(1);
  return function F() {
    // 如果被 new 创建实例，不会被改变上下文！
    if (this instanceof F) {
      return new self(...args, ...arguments);
    }
    return self.apply(context, args.concat(...arguments));
  };
};
```

## 六、深浅拷贝

### 1、浅拷贝

- 定义：拷贝原始对象的属性。如果属性为基本类型，则拷贝的是基本类型的值；如果属性是引用类型，拷贝的就是内存地址
- 特点：浅拷贝只解决了第一层的问题，拷贝第一层的基本类型值，以及第一层的引用类型地址
- 适用场景
  - Object.assign()：`let clone = Object.assign({},obj1);`
  - spread 语法：`let clone = {...object};`

### 2、深拷贝

- 定义：拷贝原对象及其引用的所有对象的所有属性值。原对象与深拷贝后的对象完全互不影响。
- 适用场景
  - `JSON.parse(JSON.stringify(oldObj))`
    - 实现了序列化与反序列化
    - 无法实现对`Function、RegExp、Date`等特殊对象的克隆
    - 忽略了 `undefined、symbol`
    - 循环引用情况下会报错
  - lodash 提供的库\_\_.cloneDeep()：以`Object.prototype.toString.call`得到`tag`，再分类处理，symbol 由`Object.getOwnPropertySymbols`方法解决
  - 手动构造：对不同的对象(正则、数组、Date 等)要采用不同的处理方式
    - 考虑数组
    - 考虑循环引用：使用`WeakMap`弱引用数据结构额外开辟一个存储空间，来存储当前对象和拷贝对象的对应关系，拷贝时先去存储空间中找，有没有拷贝过这个对象，有直接返回，没有继续拷贝。

```js
function clone(target) {
  if (typeof target === "object") {
    let cloneTarget = Array.isArray(target) ? [] : {};
    //WeakMap存储空间有拷贝过的对象则直接返回
    const map = new WeakMap();
    if (map.has(target)) {
      return map.get(target);
    }
    //WeakMap存储空间无拷贝过的对象则设置拷贝对象
    map.set(target, cloneTarget);
    //遍历被克隆对象属性
    for (const key in target) {
      //递归调用，深拷贝后依次添加到新对象上
      cloneTarget[key] = clone(target[key], map);
    }
    return cloneTarget;
  } else {
    // 原始类型，无需继续拷贝，直接返回
    return target;
  }
}
```

## 七、防抖、节流

> 防抖和节流的作用：因为无法控制 DOM 事件触发的频率，所以需在事件触发和函数执行之间添加一个控制层，防止函数高频调用。

### 1、防抖(debounce)

- 定义：事件高频触发合并为最后一次触发，触发n秒后再执行回调函数，若n秒内又被触发，则重新计时
  - 延迟执行(trailing=true)：lodash 默认开启，事件被触发 n 秒（延迟）后再执行函数调用
  - 立即执行(Leading=true)
- 适用场景：
  - input 输入控制：用户连续输入一串字符后，只会在输入完后去执行最后一次的查询ajax请求，有效减少请求次数，节约请求资源
  - window的resize、scroll事件：不断地调整浏览器的窗口大小、或者滚动时会触发对应事件，防抖让其只触发一次
- 模拟实现

```js
// immediate 表示第一次是否立即执行
function debounce(fn, wait = 50, immediate) {
  //定义 timer 定时器
  let timer = null;
  return function(...args) {
    //持续触发时不执行，清除上一次执行时的定时器
    if (timer) clearTimeout(timer);
    //选择无延迟立即执行
    if (immediate && !timer) {
      fn.apply(this, args);
    }
    //定时器重新设置计时
    timer = setTimeout(() => {
      fn.apply(this, args);
    }, wait);
  };
}
// Demo:执行 debounce 函数返回新函数
const betterFn = debounce(() => console.log("fn 防抖执行了"), 1000, true);
// 第一次触发 scroll 执行一次 fn，后续只有在停止滑动 1 秒后才执行函数 fn
document.addEventListener("scroll", betterFn);
```

### 2、节流(throttle)

- 定义：某单位时间内只能有一次触发事件的回调函数执行。
- 适用场景：
  - mousemove 事件：鼠标连续不断地触发某事件（如点击），单位时间内只触发一次
  - 页面的无限加载：需要用户在滚动页面时，每隔一段时间发一次 ajax 请求，而不是在用户停下滚动页面操作时才去请求数据
  - 监听滚动事件：比如是否滑到底部自动加载更多，用throttle来判断
- 模拟实现

```js
// fn 是需要节流处理的函数,wait 是时间间隔
function throttle(fn, wait) {
  // previous 是上一次执行 fn 的时间,timer 是定时器
  let previous = 0,
    timer = null;
  return function(...args) {
    // 获取当前时间，转换成时间戳，单位毫秒
    let now = +new Date();
    // 判断上次触发的时间和本次触发的时间差是否小于时间间隔
    if (now - previous < wait) {
      // 定时器时间结束后执行函数 fn
      if (timer) clearTimeout(timer);
      timer = setTimeout(() => {
        previous = now;
        fn.apply(this, args);
      }, wait);
    } else {
      // 第一次执行
      // 或者时间间隔超出了设定的时间间隔，执行函数 fn
      previous = now;
      fn.apply(this, args);
    }
  };
}
```

## 八、高阶函数

- 定义：高阶函数是一个接收**函数作为参数传递**或者将**函数作为返回值输出**的函数
- 函数作为参数传递：如 Array.prototype.map/filter/reduce
  - map(fn(currentValue,index,array))：每个数组元素调用函数fn，返回各自数据结果的数组
  - filter(fn(element,index,array))：返回仅通过测试函数fn的数组元素的新数组
  - reduce(fn(acc,currentValue,index,array),initial)：每个数组元素调用fn后结果存入累加器acc，累加完成后返回汇总单值
- 函数作为返回值输出
  - isType 函数：将 obj => { ... } 这一函数作为返回值输出

  ```js
  let isType = type => obj => {
    return Object.prototype.toString.call(obj) === '[object ' + type + ']';
  }

  isType('String')('123');// true
  isType('Array')([1, 2, 3]);// true
  isType('Number')(123);// true
  ```

## 九、Proxy 代理器

- 定义：Proxy 可以用来自定义对象中的操作，实现数据绑定和监听
- 语法：

```js
let p = new Proxy(target, handler);
// `target` 代表需要添加代理的对象
// `handler` 用来自定义对象中的操作
```

## 十、Promise相关

### 1、async

- 语法：async function someName(){...}
- 表示：将常规函数转换成Promise，返回值也是一个Promise对象
- 特点：async函数内部的异步操作执行完，才会执行then方法指定的回调函数

### 2、await

- 语法：[return_value] = await expression;
  - 表示：强制其他代码等待一个 promise对象或其他，直到Promise resolve完成返回result结果
- 特点：只能在async函数内部使用
- 错误处理：try{await}catch{}捕捉rejected数据或抛出异常

```js
async function f1() {
  let promise = new Promise((resolve, reject) => {
    setTimeout(() => resolve("done!"), 1000)
  });
  let result = await promise; // 等待直到 promise 决议 (*)
  alert(result); // "done!"
}
f1();
async function f2() {
  try {
    let response = await fetch('/no-user-here');
    let user = await response.json();
  } catch(err) {
    // 捕获到 fetch 和 response.json 中的错误
    alert(err);
  }
}
f2();
// 等待多个 promise 结果
let results = await Promise.all([
  fetch(url1),
  fetch(url2),
  ...
]);
```
