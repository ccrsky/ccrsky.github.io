---
title: javascript对象
date: 2016-02-11 18:44:09
tags:
    - js

category: 
    - js
---

#### 创建对象

```js
// 使用Object创建对象
var p = new Object();

p.name = 'jason';
p.sayName = function(){
    alert(this.name);
}

//使用字面量的方式创建对象
var p = {
    name:'jason',
    sayName：function(){
        alert(this.name);
    }
}
```

#### 属性类型

ECMAScript中有两种属性：数据属性，访问属性。

数据属性：是一个包含数据值的位置，有4个描述其行为的特性:

| 名称 |  值  |
| --- | ---  |
|configurable | 表示能否通过delete删除属性而重新定义，默认true | 
|enumerable | 表示能否通过for-in循环返回属性，默认true |
|writable | 表示能否修改属性的值，默认true |
|value | 包含这个属性的属性值，默认undefined |


```js
/*修改属性的默认特性，使用Object.defineProperty(obj,propertyName,descriptor)方法
四个属性值：configurable、enumerable、writable、value
*/
var p = {};
Object.defineProperty(p,'name',{
    writable:false,
    enumerable:false,
    configurable:false,
    value:'chen'
});

p.name = 'jason';  // 非严格模式下，赋值会被忽略，严格模式，赋值会报错

/*定义多个属性使用Object.definePropertys()方法 */
Object.definePropertys(p, {
  'name', {
      writable: false,
      enumerable: false,
      configurable: false,
      value: 'chen'
  }, {
      'age', {
          writable: false,
          enumerable: false,
          configurable: false,
          value: '18'
      }
  });
```

访问器属性：访问器属性不包含属性值，它们包含getter,setter函数（非必须）
该访问属性也有4个特性:

| 名称 |  值  |
| --- | ---  |
|configurable | 表示能否通过delete删除属性而重新定义，默认true|
|enumerable | 表示能否通过for-in循环返回属性，默认true |
|get|读取属性调用的函数，默认undefined|
|set|设置属性调用的函数，默认undefined|

```js
var book = {
    _year:2016,  // 私有属性常以 '_' 开头
    edition:1
};


Object.defineProperty(book,'year',{
    get:function(){
        return this._year;
    },
    set:function(newValue){
        this._year = newValue;
        this.edition +=1;
    }
});

使用访问器属性的常见方式即设置一个属性值会导致其他属性发生变化

另一种设置方式(遗弃的方法)
book.__defineGetter('year',function(){
    return this._year;
});
book.__defineSetter('year',function(newValue){
    this._year = newValue;
});

/* 读取属性的特性 Object.getOwePropertyDescriptor() 方法会返回描述对象 */
var descriptor = Object.getOwePropertyDescriptor(book,'year');

/* 返回 descriptor : {
   value:'jason',
   configurable:false
} */
```

#### 创建对象的几种方式

##### 工厂模式

```js
function createGirlFried(name,job){
    var o = new Object();
    o.name = name;
    o.job = job;
    return o;
}

var girl1 = createGirlFried('shany','Dotor');
var girl1 = createGirlFried('kaer','Softwarn Engineer');

//没有解决对象识别的问题（对象的类型）
```

#####构造函数
```js
function Person(name,job){
    this.name = name;
    this.job = job;
    this.sayName = function(){
        alert(this.name);
    }
}

var p1 = new Person('zs','doctor');
var p2 = new Person('zs','doctor');

alert(p1.construtor == p2.construtor)  // true
p1 instanceof Person  // true
p1 instanceof Object  // true
```

使用 new 操作符，调用构造函数创建对象，会经历以下4步
1. 创建一个新对象
2. 将构造函数的作用域赋给新对象（this 指向这个新对象）
3. 执行构造函数中的代码（为这个对象添加新属性）
4. 返回新对象

```js
//构造函数 当函数用
var p = Person('sy','你猜');
alert(window.name) // sy

// 构造函数的问题：不同示例上的同名函数是不相等的
function sayName (){
    alert(this.name);
}

function Person(name,job){
    this.name = name;
    this.job = job;
    this.sayName = sayName;
}
// 虽然函数一样，但污染了全局变量
```

##### 原型模式
每个函数都有一个prototype属性，这是属性是一个指针，指向一个对象（即原型对象，包含共享的属性和方法）

```js
function Person(name,job){
    this.name = name;
    this.job = job;
}

Person.prototype.sayName = function(){
    alert(this.name);
}

Person.prototype.name = 'susan';

var p1 = new Person('san','programmer');
p1._proto_ == Person.prototype // true

p1.name == san // true
```
虽然可以通过对象访问保存在原型中的值，却不能通过对象实例重写原型中的值，如果添加同名属性将屏蔽原型中的同名属性
    如果依旧要访问原型的的同名属性，只能`delete obj.propertyName`  (设置null  undefined 无效)
    
    
```js
Person.prototype.construtor == Person // true
Person.prototype.isPrototypeOf(Person) // true

检查一个属性是否是原型属性还是实例属性
hasOwnProperty('propName');
p1.hasOwnProperty('name') // true


// 遍历对象-两种方式使用in操作符( in / for in )
'name' in p1

for (var prop in p1) {
    console.log(prop)
}

/* 使用for - in返回所有能通过对象访问的、 可枚举属性（ 包含实例属 获取对象上所有可枚举的实例属性， 可使用ECMAScript5 的 Object.keys() 方法 Object.keys(p) 返回一个数组（ 顺序和for - i 获取对象上所有可枚举的实例属性， 无论是否可枚举 Object.getOwePropertyNames() ps： construtor 是不可枚举的。*/

// 重写原型对象
Person.prototype = {
        construtor: Person, // 虽说修正了 construtor 但此时的 construtor变成可枚举的，原生的construtor是不可枚举的
        sayName: function() {
            return this.name;
        }
    }

// 修复可枚举特性
Object.defineProperty(Person.prototype, 'construtor', {
    enumerable: false
})

/* 原型的问题： 原型的最大问题是由其共享属性的本性导致的， 原型中的所有属性都是被实例共享， 这种共享对函数非常合适， 对包含基本数据类型的属性也说的过去， 然而对于引用问题就比较突出了 */

Person.prototype.friends = ['sam', 'shany'];
// friends 被所有实例共享， 修改某个实例会影响其他实例

```
##### 组合使用构造函数和原型模式
```js
function person(name) {
    this.name = name;
    this.friends = ['san'];
}
Person.prototype.sayName = function() {
    alert(this.name);
}
```

##### 动态原型模式
```js
function person(name) {
    this.name = name;
    this.friends = ['san'];
    if (typeof this.sayName != 'function') {
        Person.prototype.sayName = function() {
            alert(this.name);
        }
    }
}
```
##### 寄生构造函数
```js
function Person(name, age) {
    var o = Object();
    o.name = name;
    o.age = age;
    return o;
}

var p = new Person('sam', 20);
// return 语句重写了构造函数的值

// 这个方式可在特殊情况下为对象创建构造函数

function MyArray() {
    var arr = new Array();
    arr.push.apply(arr, arguments);
    arr.print = function() {
        return this.join('-');
    }
    return arr;
}

var c = new MyArray('a1', 'a2');
alert(c.print());
```

##### 稳妥构造函数
```js
// 适合在一些安全的环境中， 没有公共属性， 其方法也不应用this

function Person(name, age) {
    var o = new Object();
    o.sayName = function() {
        alert(name);
    }
}
```

##### 原型继承
```js
function SuperType(name) {
    this.name = name;
}

function SubType(name, age) {
    SuperType.call(this, name)
    this.age = age;
}
```

##### 组合继承
```js
// 常用的继承模式（ 原型继承 父类方法无法继承过来）

function SubType(name, age) {
    SuperType.call(this, name)
    this.age = age;
}

SubType.prototype = new SuperType();
```

##### 原型继承
```js
// ECMAScript实现
function object(o) {
    function F() {}
    F.prototype = o;
    return new F();
}

// ECMAScript中 Object.create(p,{
    skill: 'js'
});
```


