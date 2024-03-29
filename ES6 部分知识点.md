# ES6 部分知识点

#### Module

模块化是个进行了很久的话题，发展历程中出现过很多模式，例如 AMD, CommonJS 等等。

`Module` 是 ES6 的新特性，是语言层面对模块化的支持。

> 与之前模块加载机制不同，Module 是动态的加载，导入的是变量的 **只读引用** ，而不是拷贝

```
// 1. export default 可以做默认导出// a.jsexport default 5;      // 默认导出// b.jsimport b, {a} from './a.js';    // 默认导入，不需要加花括号// 2. 动态的加载机制// a.jsexport let a = 10;
export let b = 10;
export function add() {
  a = 15;
  b = 20;  return a+b;
};// b.jsimport {a, b, add} from './a.js';
a+b;    // 20add();  // 35a+b;    // 35
```

#### Symbol

`symbol` 是 ES6 的一个新特性，他有如下特点：

- `symbol` 是一个 “新” 的 **基础数据类型** ；从 ES6 起，JavaScript 的 **基础数据类型** 变为 6 个：`string`, `number`, `boolean`, `null`, `undefined`, `symbol`
- `symbol` 可以用作 `Object` 的 key
- `symbol` 存在全局作用域，利用 `Symbol.for(key)` 方法，可以创建（全局作用域无指定键）或获取全局作用域内的 `symbol` ；利用 `Symbol.keyFor(sym)` 方法，可以获取指定 `symbol` 的键
- JavaScript 内部使用了很多内置 `symbol` ，作为特殊的键，来实现一些内部功能；例如 `Symbol.iterator` 用于标示对象的迭代器

> “新” 仅仅是针对前端开发人员来说的，其实 Symbol 概念本身已经在 JavaScript 语言内部长时间使用了

```
// 1. "Symbol(desc)" 方法用于创建一个新的 symbol，参数 "desc" 仅用做 symbol 的描述，并不用于唯一标示 symbolSymbol('abc') === Symbol('abc');    // false，'abc'仅作为两个 symbol 的描述信息// 2. "Symbol.for(key)" 方法，参数 "key" 是用于在全局作用域中标示 symbol 的唯一键，同时也作为该 symbol 的描述信息Symbol.for('abc') === Symbol.for('abc');    // true，左侧为创建，右侧为获取Symbol.for('abc') === Symbol('abc');        // false// 3. symbol 无法被 for...in 遍历到 (不可枚举)，可以利用 Object.getOwnPropertySymbols 获取const obj = {
  [Symbol('abc')]: 'abc',  'abc': 'abc'};for(var i in obj){  // abc
  console.log(i);
}Object.getOwnPropertySymbols(obj);    // [Symbol(abc)]
```

#### Iterator + For..Of

ES6 中除了新特性外，还有一个新的规范，那就是关于迭代的规范，他包括两部分分别是 “可迭代规范（iterable protocol）” 和 “迭代器规范（iterator protocol）”。任何实现了前者的对象，都可以进行 `for…of` 循环。

`String`, `Array`, `Map`, `Set`等是原生可迭代对象，因为他们都在原型（prototype）对象中实现了 Symbol.iterator 键对应的方法

> `for…of` 是对象迭代器的遍历，而 `for…in` 是对象中 可枚举 值的遍历

下面用代码来解释一下两个规范：

```
// 1. 迭代器规范const iter = {
  counter: 0,
  next(){    // 迭代器是实现了 "next()" 函数的对象
    if(++this.counter < 10){      return {    // 返回一个含有两个键值对的对象，Object {done => boolean, value => any}
        done: false,
        value: this.counter
      }
    }else{      this.counter = 0;      return {    // done = true 时，value非必须
        done: true
      }
    }
  }
};// 2. 可迭代规范，实现 "Symbol.iterator => func()" 键值对；而 "func()" 返回一个 迭代器对象const iterObj = {};for(var i of iterObj){};    // TypeError: iterObj is not iterableiterObj[Symbol.iterator] = function() {  return iter;
};for(var i of iterObj){  console.log(i);    // 1,2,3,4,5,6,7,8,9};
```

### 关于集合

原来我们使用集合，多数情况下会直接用 `Object` 代替，ES6新增了两个特性，`Map` 和 `Set`，他们是对 JavaScript 关于集合概念的补充。

#### Map

刚刚看到这个概念的同学会有几个常见的疑问，为什么我们需要 `Map` 这种数据结构？直接用 `Object` 不好么？ 是不是 `Map` 可以完全取代 `Object` 用于数据存取？

`Map` 与 `Object` 的区别

- `Map` 与 `Object` 都可以存取数据，`Map` 适用于存储需要 **常需要变化（增减键值对）或遍历** 的数据集，而 `Object` 适用于存储 **静态** （例如配置信息）数据集
- `Object` 的 key 必须是 `String` 或 `Symbol` 类型的，而 `Map` 无此限制，可以是任何值
- `Map` 可以很方便的取到键值对数量，而 `Object` 需要用额外途径

```
// 1. Map 的构造函数可以传入一个 “可迭代的对象（例如数组）”，其中包含键值对数组const first = new Map([['a', 1], [{'b': 1}, 2]]);    // Map {"a" => 1, Object {b: 1} => 2}// 2. Map 的键可以是任何值，甚至是 undefined 或 nullfirst.set(null, 1).set(undefined, 0);    // Map {"a" => 1, Object {b: 1} => 2, null => 1, undefined => 0}
```

#### Set

`Set` 作为最简单的集合，有着如下几个特点：

- `Set` 可以存储任何类型的值，遍历顺序与 **插入顺序相同**
- `Set` 内无重复的值

```
// 1. Set 的构造函数可以传入一个 “可迭代的对象（例如数组）”，其中包含任意值const first = new Set(['a', 1, {'b': 1}, null]);    // Set {"a", 1, Object {b: 1}, null}// 2. Set 无法插入重复的值first.add(1);    // Set {"a", 1, Object {b: 1}, null}
```

#### WeakMap + WeakSet

`WeakMap` 与 `WeakSet` 作为一个比较新颖的概念，其主要特点在于弱引用。

相比于 `Map` 与 `Set` 的强引用，弱引用可以令对象在 “适当” 情况下正确被 GC 回收，减少内存资源浪费。

**但由于不是强引用，所以无法进行遍历或取得值数量，只能用于值的存取（WeakMap）或是否存在值得判断（WeakSet）**

> 在弱引用的情况下，GC 回收时，不会把其视作一个引用；如果没有其他强引用存在，那这个对象将被回收

```
// 1. WeakMap 键必须是对象const err = new WeakMap([['a',1]]);    // TypeError: Invalid value used as weak map key// 2. WeakMap/WeakSet 的弱引用const wm = new WeakMap([[{'a':1},1]]);    // Object {'a': 1} 会正常被 GC 回收const ws = new WeakSet();
ws.add({'a':1});    // Object {'a': 1} 会正常被 GC 回收const obj = {'b': 1};
ws.add(obj);        // Object {'b': 1} 不会被正常 GC 回收，因为存在一个强引用obj = undefined;    // Object {'b': 1} 会正常被 GC 回收
```

### 异步编程

在 ES6 之前，JavaScript 的异步编程都跳不出回调函数这个方式。回调函数方式使用非常简单，在简单异步任务调用时候没有任何问题，但如果出现复杂的异步任务场景时，就显得力不从心了，最主要的问题就是多层回调函数的嵌套会导致代码的横向发展，难以维护；ES6 带来了两个新特性来解决异步编程的难题。

```
// 一个简单的多层嵌套回调函数的例子 (Node.js)const git = require('shell').git;const commitMsg = '...';

git.add('pattern/for/some/files/*', (err) => {  if(!err){
    git.commit(commitMsg, (err) => {      if(!err){
        git.push(pushOption);
      }else{        console.log(err);
      }
    });
  }else{    console.log(err);
  }
});
```

#### Promise

`Promise` 是 ES6 的一个新特性，同为异步编程方式，它主要有如下几个特点：

- 本质还是回调函数
- 区分成功和失败的回调，省去嵌套在内层的判断逻辑
- 可以很轻松的完成回调函数模式到 Promise 模式的转化
- 代码由回调函数嵌套的横向扩展，变为链式调用的纵向扩展，易于理解和维护

`Promise` 虽然优势颇多，但是代码结构仍与同步代码区别较大

```
// 上例用 Promise 实现// 假定 git.add, git.commit, git.push 均做了 Promise 封装，返回一个 Promise 对象const git = require('shell').git;const commitMsg = '...';

git.add('pattern/for/some/files/*')
  .then(() => git.commit(commitMsg))
  .then(git.push)
  .catch((err) => {    console.log(err);
  });
```

#### Generator

`Generator` 作为 ES6 的新特性，是一个语言层面的升级。它有以下几个特点：

- 可以通过 `yield` 关键字，终止执行并返回（内到外）
- 可以通过 `next(val)` 方法调用重新唤醒，继续执行（外回内）
- 运行时（包括挂起态），共享局部变量
- `Generator` 执行会返回一个结果对象，结果对象本身既是迭代器，同时也是可迭代对象（同时满足两个迭代规范），所以 `Generator` 可以直接用于 **自定义对象迭代器**

由于具备以上特点（第四点除外），`Generator` 也是 JavaScript 对 协程（coroutine）的实现，协程可以理解为 “可由开发人员控制调度的多线程”

> 协程按照调度机制来区分，可以分为对称式和非对称式
>
> 非对称式：被调用者（协程）挂起时，必须将控制权返还调用者（协程）
>
> 对称式：被调用者（协程）挂起时，可将控制权转给 “任意” 其他协程
>
> JavaScript 实现的是 非对称式协程（semi-coroutine）；非对称式协程相比于对称式协程，代码逻辑更清晰，易于理解和维护

协程给 JavaScript 提供了一个新的方式去完成异步编程；由于 `Generator` 的执行会返回一个迭代器，需要手动去遍历，所以如果要达到自动执行的目的，除了本身语法外，还需要实现一个执行器，例如 **TJ 大神的 co** 框架。

```
// 上例用 Generator 实现// 假定 git.add, git.commit, git.push 均做了 Promise 封装，返回一个 Promise 对象const co = require('co');const git = require('shell').git;

co(function* (){  const commitMsg = '...';      // 共享的局部变量

  yield git.add('pattern/for/some/files/*');  yield git.commit(commitMsg);  yield git.push();
}).catch((err) => {  console.log(err);
});
```

`Generator` 是一个 ES6 最佳的异步编程选择么？显然不是，因为除了基本语法外，我们还要额外去实现执行器来达到执行的目的，但是它整体的代码结构是优于回调函数嵌套和 `Promise` 模式的。

#### Async-Await

这并不是一个 ES6 新特性，而是 ES7 的语法，放在这里是因为它将是 JavaScript 目前支持异步编程最好的方式

```
// 上例用 async-await 实现// 假定 git.add, git.commit, git.push 均做了 Promise 封装，返回一个 Promise 对象const git = require('shell').git;

(async function(){  const commitMsg = '...';      // 共享的局部变量

  try{
    await git.add('pattern/for/some/files/*');
    await git.commit(commitMsg);
    await git.push();
  }catch(err){    console.log(err);
  }
})();
```

### 元编程

元编程是指的是开发人员对 “语言本身进行编程”。一般是编程语言暴露了一些 API，供开发人员来操作语言本身的某些特性。ES6 两个新特性 `Proxy` 和 `Reflect` 是 JavaScript 关于对象元编程能力的扩展。

#### Proxy

`Proxy` 是 ES6 加入的一个新特性，它可以 “代理” 对象的原生行为，替换为执行自定义行为。

这样的元编程能力使得我们可以更轻松的扩展出一些特殊对象。

- 任何对象都可以被 “代理”
- 利用 `Proxy.revocable(target, handler)` 可以创建出一个可逆的 “被代理” 对象

```
// 简单 element 选择控制工具的实现const cacheElement = function(target, prop) {  if(target.hasOwnProperty(prop)){      return target[prop];
    }else{      return target[prop] = document.getElementById(prop);
    }
}const elControl = new Proxy(cacheElement, {
  get: (target, prop) => {    return cacheElement(target, prop);
  },
  set: (target, prop, val) => {
    cacheElement(target, prop).textContent = val;
  },
  apply: (target, thisArg, args) => {    return Reflect.ownKeys(target);
  }
});

elControl.first;     // div#firstelControl.second;    // div#secondelControl.first = 5;    // div#first => 5elControl.second = 10;  // div#second => 10elControl();    // ['first', 'second']
```

#### Reflect

ES6 中引入的 `Reflect` 是另一个元编程的特性，它使得我们可以直接操纵对象的原生行为。`Reflect` 可操纵的行为与 `Proxy` 可代理的行为是一一对应的，这使得可以在 `Proxy` 的自定义方法中方便的使用 `Reflect` 调起原生行为。

```
// 1. Proxy 的自定义方法中，通过 Reflect 调用原生行为const customProxy = new Proxy({  'custom': 1}, {
  get: (target, prop) => {    console.log(`get ${prop} !`);    return Reflect.get(target, undefined, prop);
  }
});

customProxy.custom;  // get custom, 1// 2. 与 Object 对象上已经开放的操作原生行为方法相比，语法更加清晰易用（例如：Object.hasOwnProperty 与 Reflect.has）const symb = Symbol('b');const a = {
  [symb]: 1,  'b': 2};if(Reflect.has(a, symb) && Reflect.has(a, 'b')){  // good
  console.log('good');
}
Reflect.ownKeys(a);  // ["b", Symbol(b)]
```

## 进阶阅读

篇幅有限，无法面面俱到，还想再最后推荐给大家一些想进阶了解 ES6 的必看内容

- 如果你关注兼容性，推荐看：https://kangax.github.io/compat-table/es6/，这里介绍了从 ES5 到 ES2016+ 的所有特性（包括仍未定稿的特性）及其在各环境的兼容性
- 如果你关注性能，推荐看：http://kpdecker.github.io/six-speed/，这里通过性能测试，将 ES6 特性的原生实现与 ES5 polyfill 版本进行对比，覆盖了各主流环境；同时也可以侧面对比出各环境在原生实现上的性能优劣
- 如果你想全面了解特性，推荐看：https://developer.mozilla.org/en-US/docs/Web/JavaScript，覆盖特性的各方面，包括全面的 API（包括不推荐和废弃的）和基础用法
- 如果你想看特性更多的使用示例和对应的 polyfill 实现，推荐看：http://es6-features.org/#Constants，这里对各个特性都给出了使用丰富的例子和一个 polyfill 实现，简单明了
- 如果想了解 ECMA Script 最多最全面的细节，英语又比较过硬，推荐在需要时看：http://www.ecma-international.org/publications/files/ECMA-ST/Ecma-262.pdf，（或者直接看最新的：https://tc39.github.io/ecma262/）

## 结语

基础篇+进阶篇基本介绍完了 ES6 的主要特性，但 ES6 仅仅是现在时，后续如果大家觉得这个系列有意思，可以再写一写 ES 2016+ 的相关内容，来拥抱一下更新的变化。

最后，希望文章中的部分内容可以对大家理解和使用 ES6 有所帮助，感谢支持~

