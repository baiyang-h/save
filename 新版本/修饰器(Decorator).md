### 装饰器Decorator
`@Decorator`会自动在编译阶段执行装饰器函数, 如果是`@Decorator(args)`则是执行`@Decorator`返回值的()。  

这样就相当于这个要被修饰的函数，被重新定义了，重新修饰了。

#### 类的修饰
许多面向对象的语言都有修饰器（`Decorator`）函数，用来修改类的行为。
```js
@testable
class MyTestableClass {
  // ...
}
function testable(target) {
  target.isTestable = true;
  target.prototype.aa = true;
}

MyTestableClass.isTestable // true
let obj = new MyTestableClass();
obj.aa // true
```
`@testable`就是一个修饰器。它修改了`MyTestableClass`这个类的行为，为它加上了静态属性`isTestable`。`testable`函数的参数`target`是`MyTestableClass`类本身。

基本上，修饰器的行为就是下面这样。
```js
@decorator
class A {}
// 等同于
class A {}
A = decorator(A) || A;
```
也就是说，修饰器是一个对类进行处理的函数。修饰器函数的第一个参数，就是所要修饰的目标类。

如果觉得一个参数不够用，可以在修饰器外面再封装一层函数。
```js
function testable(isTestable) {
  return function(target) {
    target.isTestable = isTestable;     //没有return 默认return为undefined
  }
}

@testable(true)
class MyTestableClass {}
MyTestableClass.isTestable // true

@testable(false)
class MyClass {}
MyClass.isTestable // false
```
上面代码中，修饰器testable可以接受参数，这就等于可以修改修饰器的行为。

**注意，修饰器对类的行为的改变，是代码编译时发生的，而不是在运行时。这意味着，修饰器能在编译阶段运行代码。也就是说，修饰器本质就是编译时执行的函数。**

例子
```js
// mixins.js
export function mixins(...list) {
  return function (target) {
    Object.assign(target.prototype, ...list)
  }
}

// main.js
import { mixins } from './mixins'

const Foo = {
  foo() { console.log('foo') }
};

@mixins(Foo)
class MyClass {}

let obj = new MyClass();
obj.foo() // 'foo'
```

#### 方法的修饰
修饰器不仅可以修饰类，还可以修饰类的属性。
```js
class Person {
  @readonly
  name() { return `${this.first} ${this.last}` }
}

function readonly(target, name, descriptor){
  // descriptor对象原来的值如下
  // {
  //   value: specifiedFunction,
  //   enumerable: false,
  //   configurable: true,
  //   writable: true
  // };
  descriptor.writable = false;
  return descriptor;
}

readonly(Person.prototype, 'name', descriptor);
// 类似于
Object.defineProperty(Person.prototype, 'name', descriptor);
```
修饰器第一个参数是类的原型对象，第二个参数是所要修饰的属性名，第三个参数是该属性的描述对象。

另外，上面代码说明，修饰器（readonly）会修改属性的描述对象（descriptor），然后被修改的描述对象再用来定义属性。

例子1
```js
class Math {
  @log
  add(a, b) {
    return a + b;
  }
}

function log(target, name, descriptor) {
  var oldValue = descriptor.value;

  descriptor.value = function() {
    console.log(`Calling ${name} with`, arguments);
    return oldValue.apply(this, arguments);
  };

  return descriptor;
}

const math = new Math();

// passed parameters should get logged now
math.add(2, 4);
```

例子2，修饰器里传递参数
```js
class Math {
  @log(1, 2)
  add(a, b) {
    return a + b;
  }
}

function log(q, w) {
    //do something
    return function(target, name, descriptor) {
        var oldValue = descriptor.value;
        descriptor.value = function() {
            console.log(`Calling ${name} with`, arguments);
            return oldValue.apply(this, arguments);
        };
        return descriptor;
    }
}

const math = new Math();

// passed parameters should get logged now
math.add(2, 4);
```



**注意：修饰器只能用于类和类的方法，不能用于函数，因为存在函数提升。**

参考文章
[阮老师的es6](http://es6.ruanyifeng.com/#docs/decorato)
[ES7 Decorator 装饰者模式](http://taobaofed.org/blog/2015/11/16/es7-decorator/)