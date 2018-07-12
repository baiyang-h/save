### Symbol
`ES5` 的对象属性名都是字符串，这容易造成属性名的冲突。`ES6` 引入了一种新的原始数据类型`Symbol`，表示独一无二的值。

现在`JavaScript`存在7种数据类型，
- `undefined`
- `null`
- `布尔值(Boolean)`
- `字符串(String)`
- `数值(Number)`
- `对象(Object)`
- `Symbol`


#### 1.概述

##### 创建Symbol
`Symbol` 值通过`Symbol`函数生成。
```js
let s = Symbol();

typeof s
// "symbol"         s是独一无二的
```

注意，`Symbol`函数前不能使用`new`命令，否则会报错。这是因为生成的 `Symbol` 是一个原始类型的值，不是对象。也就是说，由于 `Symbol` 值不是对象，所以不能添加属性。基本上，它是一种类似于字符串的数据类型。

##### Symbol传入参数
`Symbol`函数可以接受一个**字符串作为参数**，表示对 `Symbol` 实例的描述，主要是为了在控制台显示，或者转为字符串时，比较容易区分。
```js
let s1 = Symbol('foo');
let s2 = Symbol('bar');

s1 // Symbol(foo)
s2 // Symbol(bar)

s1.toString() // "Symbol(foo)"
s2.toString() // "Symbol(bar)"
```

如果 `Symbol` 的参数是一个对象，就会调用该对象的`toString`方法，将其转为字符串，然后才生成一个 `Symbol` 值。
```js
const obj = {
  toString() {
    return 'abc';
  }
};
const sym = Symbol(obj);
sym // Symbol(abc)
```
##### 注意事项
`Symbol`函数的参数只是表示对当前 `Symbol` 值的描述，因此相同参数的`Symbol`函数的返回值是不相等的。
```js
// 没有参数的情况
let s1 = Symbol();
let s2 = Symbol();

s1 === s2 // false

// 有参数的情况
let s1 = Symbol('foo');
let s2 = Symbol('foo');

s1 === s2 // false
```

`Symbol` 值不能与其他类型的值进行运算，会报错。
```js
let sym = Symbol('My symbol');

"your symbol is " + sym
// TypeError: can't convert symbol to string
`your symbol is ${sym}`
// TypeError: can't convert symbol to string
```
但是，`Symbol` 值可以显式转为字符串。
```js
let sym = Symbol('My symbol');

String(sym) // 'Symbol(My symbol)'
sym.toString() // 'Symbol(My symbol)'
```
另外，`Symbol` 值也可以转为布尔值，但是不能转为数值。
```js
let sym = Symbol();
Boolean(sym) // true
!sym  // false

if (sym) {
  // ...
}

Number(sym) // TypeError
sym + 2 // TypeError
```

#### 2.作为属性名的 Symbol
```js
let mySymbol = Symbol();

// 第一种写法
let a = {};
a[mySymbol] = 'Hello!';

// 第二种写法
let a = {
  [mySymbol]: 'Hello!'
};

// 第三种写法
let a = {};
Object.defineProperty(a, mySymbol, { value: 'Hello!' });

// 以上写法都得到同样结果
a[mySymbol] // "Hello!"
```
注意，`Symbol` 值作为对象属性名时，不能用点运算符。

#### 3.消除魔术字符串
```js
//在switch语法中的用处
const COLOR_RED    = Symbol();
const COLOR_GREEN  = Symbol();

function getComplement(color) {
  switch (color) {
    case COLOR_RED:
      return COLOR_GREEN;
    case COLOR_GREEN:
      return COLOR_RED;
    default:
      throw new Error('Undefined color');
    }
}
```
例子
```js
function getArea(shape, options) {
  let area = 0;

  switch (shape) {
    case 'Triangle': // 魔术字符串
      area = .5 * options.width * options.height;
      break;
    /* ... more code ... */
  }

  return area;
}

getArea('Triangle', { width: 100, height: 100 }); // 魔术字符串
```
其中的`Triangle`这个字符串形成了高度耦合。

常用的消除魔术字符串的方法，就是把它写成一个变量。
```js
const shapeType = {
  triangle: 'Triangle'
};

function getArea(shape, options) {
  let area = 0;
  switch (shape) {
    case shapeType.triangle:
      area = .5 * options.width * options.height;
      break;
  }
  return area;
}

getArea(shapeType.triangle, { width: 100, height: 100 });
```
其实`shapeType.triangle`等于哪个值并不重要，只要确保不会跟其他`shapeType`属性的值冲突即可。因此，这里就很适合改用 `Symbol` 值。
```js
//改成Symbol
const shapeType = {
  triangle: Symbol()
};
```

#### 4.属性名的遍历
`Symbol` 作为属性名，该属性不会出现在`for...in`、`for...of`循环中，也不会被`Object.keys()`、`Object.getOwnPropertyNames()`、`JSON.stringify()`返回。

但是，它也不是私有属性，有一个`Object.getOwnPropertySymbols`方法，可以获取指定对象的所有 `Symbol` 属性名, 返回的是一个数组。
```js
const obj = {};
let a = Symbol('a');
let b = Symbol('b');

obj[a] = 'Hello';
obj[b] = 'World';

const objectSymbols = Object.getOwnPropertySymbols(obj);

objectSymbols
// [Symbol(a), Symbol(b)]
```
另一个例子，`Object.getOwnPropertySymbols`方法与`for...in`循环、`Object.getOwnPropertyNames`方法进行对比的例子。
```js
const obj = {};

let foo = Symbol("foo");

Object.defineProperty(obj, foo, {
  value: "foobar",
});

for (let i in obj) {
  console.log(i); // 无输出
}

Object.getOwnPropertyNames(obj)
// []

Object.getOwnPropertySymbols(obj)
// [Symbol(foo)]
```
使用`Object.getOwnPropertyNames`方法得不到`Symbol`属性名，需要使用`Object.getOwnPropertySymbols`方法。

不过有一个新的 API，`Reflect.ownKeys`方法可以返回所有类型的键名，包括常规键名和 `Symbol` 键名。
```js
let obj = {
  [Symbol('my_key')]: 1,
  enum: 2,
  nonEnum: 3
};

Reflect.ownKeys(obj)
//  ["enum", "nonEnum", Symbol(my_key)]
```
由于以 `Symbol` 值作为名称的属性，不会被常规方法遍历得到。我们可以利用这个特性，为对象定义一些非私有的、但又希望只用于内部的方法。

#### Symbol.for()，Symbol.keyFor()
有时，我们希望重新使用同一个 `Symbol` 值，`Symbol.for`方法可以做到这一点。它接受一个字符串作为参数，然后搜索有没有以该参数作为名称的 `Symbol` 值。如果有，就返回这个 `Symbol` 值，否则就新建并返回一个以该字符串为名称的 `Symbol` 值。
```js
let s1 = Symbol.for('foo');
let s2 = Symbol.for('foo');

s1 === s2 // true
```
上面代码中，`s1`和`s2`都是 `Symbol` 值，但是它们都是同样参数的`Symbol.for`方法生成的，所以实际上是同一个值。

`Symbol.for()`与`Symbol()`这两种写法，都会生成新的 `Symbol`。它们的区别是，前者会被登记在全局环境中供搜索，后者不会。`Symbol.for()`不会每次调用就返回一个新的 `Symbol` 类型的值，而是会先检查给定的key是否已经存在，如果不存在才会新建一个值。比如，如果你调用`Symbol.for("cat")`30 次，每次都会返回同一个 `Symbol` 值，但是调用`Symbol("cat")`30 次，会返回 30 个不同的 `Symbol` 值。
```js
Symbol.for("bar") === Symbol.for("bar")
// true

Symbol("bar") === Symbol("bar")
// false
```
由于`Symbol()`写法没有登记机制，所以每次调用都会返回一个不同的值。

`Symbol.keyFor`方法返回一个已登记的 `Symbol` 类型值的`key`,即使用`Symbol.for(x)`创建的`Symbol`值。
```js
let s1 = Symbol.for("foo");
Symbol.keyFor(s1) // "foo"

let s2 = Symbol("foo");
Symbol.keyFor(s2) // undefined  s2属于未登记的 Symbol 值
```
需要注意的是，`Symbol.for`为 `Symbol` 值登记的名字，是全局环境的，可以在不同的 `iframe` 或 `service worker` 中取到同一个值。

[实例：模块的 Singleton 模式  例子](http://es6.ruanyifeng.com/#docs/symbol#属性名的遍历)

#### 内置的 Symbol 值
[查阅阮老师的es6文档](http://es6.ruanyifeng.com/#docs/symbol#内置的-Symbol-值)

<br/>
参考文档

[阮老师的Symbol](http://es6.ruanyifeng.com/#docs/symbol)