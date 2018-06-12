- [开始](#开始)
- [Object.getPrototypeOf()](#getPrototypeOf)
- [super](#super)

### 开始
`Class` 可以通过`extends`关键字，继承了`Point`类的所有属性和方法。
```js
class Point {
}

class ColorPoint extends Point {
}
```

```js
class ColorPoint extends Point {
  constructor(x, y, color) {
    super(x, y); // 调用父类的constructor(x, y)
    this.color = color;
  }

  toString() {
    return this.color + ' ' + super.toString(); // 调用父类的toString() 
    //this.toString()也一样因为this已经继承刀客父类的属性和方法
  }
}
```
上面代码中，`constructor`方法和`toString`方法之中，都出现了`super`关键字，它在这里表示父类的构造函数，用来新建父类的`this`对象。

子类必须在`constructor`方法中调用`super`方法，否则新建实例时会报错。

`ES5` 的继承，实质是先创造子类的实例对象`this`，然后再将父类的方法添加到`this`上面`（Parent.apply(this)）`。`ES6` 的继承机制完全不同，实质是先创造父类的实例对象`this`（所以必须先调用`super`方法），然后再用子类的构造函数修改`this`。

如果子类没有定义`constructor`方法，这个方法会被默认添加，代码如下。
```js
class ColorPoint extends Point {
}

// 等同于
class ColorPoint extends Point {
  constructor(...args) {
    super(...args);
  }
}
```

另一个需要注意的地方是，在子类的构造函数中，只有调用`super`之后，才可以使用`this`关键字，否则会报错。

父类的静态方法，也会被子类继承。
```js
class A {
  static hello() {
    console.log('hello world');
  }
}

class B extends A {
}

B.hello()  // hello world
```

### <span id='getPrototypeOf'>Object.getPrototypeOf()</span>
`Object.getPrototypeOf`方法可以用来从子类上获取父类。 或者实例获取他的原型对象
```js
Object.getPrototypeOf(ColorPoint) === Point     // true
Object.getPrototypeOf(ColorPoint的实例) === ColorPoint.prototype   //true
```
因此，可以使用这个方法判断，一个类是否继承了另一个类。

### super
`super`这个关键字，既可以当作函数使用，也可以当作对象使用。在这两种情况下，它的用法完全不同。
1. 第一种情况，`super`作为函数调用时，代表父类的构造函数。`ES6` 要求，子类的构造函数必须执行一次`super`函数。
```js
class A {}

class B extends A {
  constructor() {
    super();
  }
}
```
注意，`super`虽然代表了父类`A`的构造函数，但是返回的是子类`B`的实例，即`super`内部的`this`指的是`B`，因此`super()`在这里相当于`A.prototype.constructor.call(this)`。

作为函数时，super()只能用在子类的构造函数之中，用在其他地方就会报错。
```js
class A {}

class B extends A {
  m() {
    super(); // 报错
  }
}
```

2. 第二种情况，super作为对象时，在普通方法中，指向父类的原型对象；在静态方法中，指向父类。
```js
class A {
  p() {
    return 2;
  }
}

class B extends A {
  constructor() {
    super();
    console.log(super.p()); // 2
  }
}

let b = new B();
```
上面代码中，子类`B`当中的`super.p()`，就是将`super`当作一个对象使用。这时，`super`在普通方法之中，指向`A.prototype`，所以`super.p()`就相当于`A.prototype.p()`。

这里需要注意，由于`super`指向父类的原型对象，所以定义在父类实例上的方法或属性，是无法通过`super`调用的。
```js
class A {
  constructor() {
    this.p = 2;
  }
}

class B extends A {
  get m() {
    return super.p;
  }
}

let b = new B();
b.m // undefined
```

ES6 规定，在子类普通方法中通过super调用父类的方法时，**方法内部的this**指向当前的子类实例。
```js
class A {
  constructor() {
    this.x = 1;
  }
  print() {
    console.log(this.x);
  }
}

class B extends A {
  constructor() {
    super();
    this.x = 2;
  }
  m() {
    super.print();    //函数内部的this是子类
  }
}

let b = new B();
b.m() // 2
```

由于`this`指向子类实例，所以如果通过`super`对某个属性赋值，这时`super`就是`this`，赋值属性会变成子类实例的属性。
```js
class A {
  constructor() {
    this.x = 1;
  }
}

class B extends A {
  constructor() {
    super();
    this.x = 2;
    super.x = 3;            //this.x = 3
    console.log(super.x); // undefined
    console.log(this.x); // 3
  }
}

let b = new B();
```
上面代码中，`super.x`赋值为3，这时等同于对`this.x`赋值为3。而当读取`super.x`的时候，读的是`A.prototype.x`，所以返回`undefined`。

如果`super`作为对象，用在静态方法之中，这时`super`将指向父类，而不是父类的原型对象。

另外，在子类的静态方法中通过`super`调用父类的方法时，方法内部的`this`指向当前的子类，而不是子类的实例。

### 类的 prototype 属性和__proto__属性
`Class` 作为构造函数的语法糖，同时有`prototype`属性和`__proto__`属性，因此同时存在两条继承链。
1. 子类的`__proto__`属性，表示构造函数的继承，总是指向父类。
2. 子类`prototype`属性的`__proto__`属性，表示方法的继承，总是指向父类的`prototype`属性。

```js
class A {
}

class B extends A {
}

B.__proto__ === A // true
B.prototype.__proto__ === A.prototype // true
```

### extends
```js
class B extends A, AA {
}
```
下面，讨论三种特殊情况。
1. 第一种特殊情况，子类继承Object类。
```js
class A extends Object {
}

A.__proto__ === Object // true
A.prototype.__proto__ === Object.prototype // true
```

2. 第二种特殊情况，不存在任何继承。
```js
class A {
}

A.__proto__ === Function.prototype // true
A.prototype.__proto__ === Object.prototype // true
```

3. 第三种特殊情况，子类继承`null`。
```js
class A extends null {
}

A.__proto__ === Function.prototype // true
A.prototype.__proto__ === undefined // true
```
这种情况与第二种情况非常像。`A`也是一个普通函数，所以直接继承`Function.prototype`。但是，`A`调用后返回的对象不继承任何方法，所以它的`__proto__`指向`undefined`，即实质上执行了下面的代码。
```js
class C extends null {
  constructor() { return Object.create(null); }
}
```

### 原生构造函数的继承
以前，原生构造函数是无法继承的。

`ES6` 允许继承原生构造函数定义子类，因为 `ES6` 是先新建父类的实例对象this，然后再用子类的构造函数修饰`this`，使得父类的所有行为都可以继承。下面是一个继承`Array`的例子。
```js
class MyArray extends Array {
  constructor(...args) {
    super(...args);
  }
}

var arr = new MyArray();
arr[0] = 12;
arr.length // 1

arr.length = 0;
arr[0] // undefined

//创建了一个数组副本
```
这意味着，`ES6` 可以自定义原生数据结构（比如`Array`、`String`等）的子类，这是 `ES5` 无法做到的。


