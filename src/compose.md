### 了解express/koa中间件及Compose函数
---

- [中间件使用方式](中间件使用方式)
- [柯里化与compose函数](柯里化与compose函数)
- [express中间件的实现](express中间件的实现)
- [koa中间件的实现](koa中间件的实现)
- [结语](结语)

#### 中间件使用方式
在express/koa中，我们会跟许许多多的中间件打交道。有第三方实现的，比如用于路由、日志、安全认证的；也有自定义的，来满足我们特定项目的需要。

首先我们来看一个使用express中间件的例子：
```javascript
const f1 = (req, res, next) => {
  console.log(1);
  next();
  console.log(6);
};
const f2 = (req, res, next) => {
  console.log(2);
  next();
  console.log(5);
};
const f3 = (req, res, next) => {
  console.log(3);
  next();
  console.log(4);
};
app.use(f1);
app.use(f2);
app.use(f3);
// output
1
2
3
4
5
6
```
观察它的输出顺序，可以发现：
```javascript
// f3的next:
() => {}
// f2的next:
() => f3(req, res, () => {}))
// f1的next:
() => f2(req, res, () => f3(req, res, () => {}))
```
合起来就是：`f1(req, res, () => f2(req, res, () => f3(req, res, () => {}))`

有点复杂，去掉req, res参数后简化为：`f1(() => f2(() => f3(() => {})))`

那么该如何去抽象出这么一个函数呢？


#### 柯里化(currying)与compose函数

- 什么是柯里化
  ```javascript
  // 柯里化前
  const fn = (a, b) => a + b;
  // 柯里化后
  const fn = a => b => a + b;
  ```
  简单来讲，柯里化就是一个高阶函数，每次只接受一个参数，返回另一个函数

- compose函数

  现在我们需要实现一个函数，逻辑为：f1(f2(f3(...)))
  ```javascript
  const compose = (...fns) => (...args) => fns.reduce((a, b) => a(b(...args)));
  // 或者 (推荐写法)
  const compose = (...fns) => fns.reduce((a, b) => (...args) => a(b(...args)));
  // 或者
  const compose = (...fns) => fns.reduceRight((a, b) => (...args) => b(a(...args)));
  ```

- 为什么要柯里化及应用场景

  通过柯里化函数和compose函数进行结合使用，可以提高复杂逻辑的可读性。[参考&nbsp;why-curry-helps.md](https://gist.github.com/jcouyang/b56a830cd55bd230049f)

  ```javascript
  const arr = [1, 2, 3];
  // 加
  const add = m => arr => arr.map(n => m + n);
  // 乘
  const multiple = m => arr => arr.map(n => m * n);
  // 拼接
  const contact = m => arr => arr.reduce((a, b) => a + m + b);

  const compose = (...fns) => fns.reduce((a, b) => (...args) => a(b(...args)));

  compose(
    contact(''),
    multiple(2),
    add(1)
  )(arr);
  // output
  6
  ```

#### express中间件的实现

express的中间件实现本质就是一个compose。

compose的第一个参数为所有的中间件，第二个参数就是(req, res)。跟上面的区别是，每个函数都有一个next，来控制函数的执行逻辑。
```javascript
const compose = (...fns) => fns.reduceRight((a, b) => (...args) => b(...args, () => a(...args)), () => {});
compose(f1, f2, f3)({}, {});
```
#### koa中间件的实现
koa的中间件逻辑和express类似，只不过中间件每次返回的是一个promise。
```javascript
const compose = (...fns) => fns.reduceRight((a, b) => (...args) => Promise.resolve(b(...args, () => a(...args))), () => {});

const f1 = async (ctx, next) => {
  console.log(1);
  await next();
  console.log(6);
};
const f2 = async (ctx, next) => {
  console.log(2);
  await next();
  console.log(5);
};
const f3 = async (ctx, next) => {
  console.log(3);
  await next();
  console.log(4);
};

compose(f1, f2, f3)({});
```

#### [结语](结语)
- [https://gist.github.com/jcouyang/b56a830cd55bd230049f](https://gist.github.com/jcouyang/b56a830cd55bd230049f)
- [https://www.codementor.io/@michelre/use-function-composition-in-javascript-gkmxos5mj](https://www.codementor.io/@michelre/use-function-composition-in-javascript-gkmxos5mj)
- [https://gist.github.com/JamieMason/172460a36a0eaef24233e6edb2706f83](https://gist.github.com/JamieMason/172460a36a0eaef24233e6edb2706f83)
