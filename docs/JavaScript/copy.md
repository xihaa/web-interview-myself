# 面试官：深拷贝浅拷贝的区别？如何实现一个深拷贝？

 ![](https://static.vue-js.com/cdf952e0-69b8-11eb-85f6-6fac77c0c9b3.png)

## 一、数据类型存储

前面文章我们讲到，`JavaScript`中存在两大数据类型：

- 基本类型
- 引用类型 

基本类型数据保存在在栈内存中

引用类型数据保存在堆内存中，引用数据类型的变量是一个指向堆内存中实际对象的引用，存在栈中



## 二、浅拷贝

浅拷贝，指的是创建新的数据，这个数据有着原始数据属性值的一份精确拷贝

如果属性是基本类型，拷贝的就是基本类型的值。如果属性是引用类型，拷贝的就是内存地址

即浅拷贝是拷贝一层，深层次的引用类型则共享内存地址

下面简单实现一个浅拷贝

```js
function shallowClone(obj) {
    const newObj = {};
    for(let prop in obj) {
        if(obj.hasOwnProperty(prop)){
            newObj[prop] = obj[prop];
        }
    }
    return newObj;
}
```

在`JavaScript`中，存在浅拷贝的现象有：

- `Object.assign`
- `Array.prototype.slice()`, `Array.prototype.concat()`
- 使用拓展运算符实现的复制





### Object.assign

```js
var obj = {
    age: 18,
    nature: ['smart', 'good'],
    names: {
        name1: 'fx',
        name2: 'xka'
    },
    love: function () {
        console.log('fx is a great girl')
    }
}
var newObj = Object.assign({}, fxObj);
```



### slice()

```js
const fxArr = ["One", "Two", "Three"]
const fxArrs = fxArr.slice(0)
fxArrs[1] = "love";
console.log(fxArr) // ["One", "Two", "Three"]
console.log(fxArrs) // ["One", "love", "Three"]
```



### concat()

```js
const fxArr = ["One", "Two", "Three"]
const fxArrs = fxArr.concat()
fxArrs[1] = "love";
console.log(fxArr) // ["One", "Two", "Three"]
console.log(fxArrs) // ["One", "love", "Three"]
```







### 拓展运算符

```js
const fxArr = ["One", "Two", "Three"]
const fxArrs = [...fxArr]
fxArrs[1] = "love";
console.log(fxArr) // ["One", "Two", "Three"]
console.log(fxArrs) // ["One", "love", "Three"]
```





## 三、深拷贝

深拷贝开辟一个新的栈，两个对象属完成相同，但是对应两个不同的地址，修改一个对象的属性，不会改变另一个对象的属性

常见的深拷贝方式有：

- _.cloneDeep()
- structuredClone()
- jQuery.extend()
- JSON.stringify()
- 手写循环递归



### _.cloneDeep()

```js
const _ = require('lodash');
const obj1 = {
    a: 1,
    b: { f: { g: 1 } },
    c: [1, 2, 3]
};
const obj2 = _.cloneDeep(obj1);
console.log(obj1.b.f === obj2.b.f);// false
```



### structuredClone() 

现代且推荐的方法：structuredClone()
对于现代浏览器和 JavaScript 环境（如 Node.js v17+），最简单且最强大的深拷贝方法是使用内置的 structuredClone() API。

```js
const original = {
  name: 'John Doe',
  age: 30,
  address: {
    street: '123 Main St',
    city: 'Anytown'
  },
  hobbies: ['reading', 'coding'],
  metadata: new Map([['key', 'value']]),
  registered: new Date()
};

const cloned = structuredClone(original);

// 修改克隆后的对象
cloned.address.city = 'New City';
cloned.hobbies.push('hiking');
cloned.metadata.set('newKey', 'newValue');

console.log('Original:', original);
console.log('Cloned:', cloned);

// 验证原始对象未被修改
console.log(original.address.city); // "Anytown"
console.log(original.hobbies); // ["reading", "coding"]
console.log(original.metadata.get('newKey')); // undefined
```
structuredClone() 的优势在于它可以处理循环引用、多种内置数据类型（如 Date, RegExp, Map, Set, ArrayBuffer 等），并且通常比手写的递归函数性能更好。但是，它不能复制函数、DOM 节点或对象的原型链。





### JSON.stringify()

```js
const obj2=JSON.parse(JSON.stringify(obj1));
```

但是这种方式存在弊端，会忽略`undefined`、`symbol`和`函数`

```js
const obj = {
    name: 'A',
    name1: undefined,
    name3: function() {},
    name4:  Symbol('A')
}
const obj2 = JSON.parse(JSON.stringify(obj));
console.log(obj2); // {name: "A"}
```



### 循环递归

```js
function deepClone(obj, hash = new WeakMap()) {
  if (obj === null) return obj; // 如果是null或者undefined我就不进行拷贝操作
  if (obj instanceof Date) return new Date(obj);
  if (obj instanceof RegExp) return new RegExp(obj);
  // 可能是对象或者普通的值  如果是函数的话是不需要深拷贝
  if (typeof obj !== "object") return obj;
  // 是对象的话就要进行深拷贝
  if (hash.get(obj)) return hash.get(obj);
  let cloneObj = new obj.constructor();
  // 找到的是所属类原型上的constructor,而原型上的 constructor指向的是当前类本身
  hash.set(obj, cloneObj);
  for (let key in obj) {
    if (obj.hasOwnProperty(key)) {
      // 实现一个递归拷贝
      cloneObj[key] = deepClone(obj[key], hash);
    }
  }
  return cloneObj;
}
```

-----

### 【插入】深入理解 JavaScript 深拷贝 (cloneDeep) 与循环引用

在 JavaScript 开发中，深拷贝（Deep Clone）是一个常见但又极具挑战性的任务。一个健壮的 `cloneDeep` 函数不仅要能复制对象和数组，还必须能优雅地处理“循环引用”这一棘手问题。

本文将通过一个生动的比喻，帮助你彻底理解循环引用是什么，以及 `cloneDeep` 函数是如何巧妙地解决这个问题的。

#### 一、循环引用到底是什么？

想象一下，你在写一本关于“人物关系”的书。

>   - 书里有一个角色叫“张三”。
>   - 在“张三”的介绍里，你写道：“他最好的朋友是‘李四’（详见第20页）”。
>   - 然后你翻到第20页，开始写“李四”的介绍。
>   - 在“李四”的介绍里，你又写道：“他最好的朋友是‘张三’（详见第10页）”。

现在，一个**没有脑子、只会按指令办事**的复印机（也就是一个简单的、未处理循环引用的 `cloneDeep` 函数）开始工作。它的任务是：“把这本书里提到的所有人物介绍，都完整地复印一份出来。”

##### 复印过程会发生什么？

1.  **复印机开始复印“张三”**。它看到“最好的朋友是‘李四’（详见第20页）”。
2.  为了完成任务，它立刻**跳转到第20页去复印“李四”**。
3.  在复印“李四”时，它看到“最好的朋友是‘张三’（详见第10页）”。
4.  为了完成任务，它又立刻**跳转回第10页去复印“张三”**。
5.  在复印“张三”时，它又看到“最好的朋友是‘李四’（详见第20页）”……
6.  复印机就这样在第10页和第20页之间**无限来回跳转**，永远无法完成工作，直到它的内存（就像复印机的纸）耗尽，程序崩溃。

这就是**循环引用**导致的问题：**无限递归（Infinite Recursion）**。

在代码中，这个“人物关系”就是对象之间的相互引用：

```javascript
let zhangSan = {
  name: '张三'
};

let liSi = {
  name: '李四'
};

// 张三的朋友是李四
zhangSan.friend = liSi;

// 李四的朋友是张三
liSi.friend = zhangSan;

// 现在 zhangSan 和 liSi 就形成了一个循环引用
console.log(zhangSan.friend.friend === zhangSan); // true
console.log(liSi.friend.friend === liSi);   // true
```

一个简陋的拷贝函数 `naiveClone(obj)` 会这样做：

  - `naiveClone(zhangSan)`
  - 发现 `friend` 属性，于是调用 `naiveClone(liSi)`
  - 在拷贝 `liSi` 时，发现 `friend` 属性，于是调用 `naiveClone(zhangSan)`
  - 在拷贝 `zhangSan` 时，又发现 `friend` 属性，于是调用 `naiveClone(liSi)`
  - ... 无限循环下去，直到“栈溢出 (Stack Overflow)”错误。

-----

#### 二、`cloneDeep` 的实现原理

现在我们知道问题所在了。如果你是一个**聪明的复印员**，而不是一个笨机器，你会怎么做？

你可能会拿一本**备忘录**。

##### 聪明的复印策略 (`cloneDeep` 的原理)：

1.  **开始复印前，准备一本空的备忘录。**

      * 在我们的代码里，这个备忘录就是 `new WeakMap()`。`WeakMap` 是一个特殊的数据结构，专门用来存储“原件 -\> 复印件”的对应关系。

2.  **开始复印“张三”**。

      * **第一步，先查备忘录**：“我复印过‘张三’吗？” 答案是“没有”。
      * **第二步，立即在备忘录里记一笔**：“我现在要开始复印‘张三’了，我先给他准备一个空的复印件。”
          * 代码实现：`hash.set(obj, cloneObj);` 这一步至关重要，它是在**深入递归之前**就记录下来。

3.  **复印“张三”的内容**。你看到“最好的朋友是‘李四’”。

      * 你需要去复印“李四”。

4.  **开始复印“李四”**。

      * **第一步，查备忘录**：“我复印过‘李四’吗？” 答案是“没有”。
      * **第二步，立即在备忘录里记一笔**：“我现在要开始复印‘李四’了，也给他准备一个空的复印件。”

5.  **复印“李四”的内容**。你看到“最好的朋友是‘张三’”。

      * 你需要去复印“张三”。

6.  **回到“张三”**。

      * **第一步，查备忘录**：“我复印过‘张三’吗？” **答案是“有！”**。备忘录里记录着你之前为“张三”准备的那个**空的复印件**。
      * **第二步，直接拿来用**。你不需要重新创建一个新的“张三”复印件，而是直接把你备忘录里存的那个“张三”的复印件拿出来用。
          * 代码实现：`if (hash.has(obj)) { return hash.get(obj); }`

7.  这样，复印“李四”的工作就完成了。它的 `friend` 属性指向了**已经创建好的“张三”复印件**。

8.  程序返回，继续完成“张三”的复印工作。它的 `friend` 属性指向了**刚刚完成的“李四”复印件**。

最终，你成功地复印了所有内容，并且复印件之间也保持了正确的“朋友关系”，没有陷入死循环。

-----

#### 三、代码实现与解析

让我们在代码中用注释标出这个“聪明的复印策略”：

```javascript
/**
 * 实现一个能处理循环引用的深拷贝函数
 * @param {any} obj - 需要被深拷贝的源对象
 * @param {WeakMap} hash - 用于存储已拷贝对象的哈希表（备忘录），解决循环引用问题
 * @returns {any} - 拷贝后的新对象
 */
function cloneDeep(obj, hash = new WeakMap()) {
  // 备忘录，用于解决循环引用
  
  // 处理 null 或非对象类型，直接返回
  if (obj === null || typeof obj !== 'object') {
    return obj;
  }
  
  // 处理特殊对象：Date 和 RegExp
  if (obj instanceof Date) {
    return new Date(obj);
  }
  if (obj instanceof RegExp) {
    return new RegExp(obj);
  }
  
  // 关键步骤1：检查备忘录
  // 在拷贝当前对象 obj 之前，先查一下备忘录里是不是已经有它的记录了？
  if (hash.has(obj)) {
    // 如果有，说明之前已经遇到过这个对象了（这就是循环引用的情况！）
    // 直接返回当时为它创建的那个“复印件”，不再继续深入，从而打断循环。
    return hash.get(obj);
  }

  // 根据源对象是数组还是对象，创建一个空的“复印件”
  const cloneObj = Array.isArray(obj) ? [] : {};

  // 关键步骤2：记入备忘录
  // 在深入拷贝它的属性之前，立刻把“原件 -> 复印件”的对应关系记下来。
  // 这样，如果在后面的递归中又遇到了 obj，上面的 if(hash.has(obj)) 就能生效。
  hash.set(obj, cloneObj);

  // 遍历源对象的属性进行递归拷贝
  for (let key in obj) {
    // 只拷贝对象自身的属性，忽略原型链上的属性
    if (Object.prototype.hasOwnProperty.call(obj, key)) {
      // 递归调用，把备忘录也传下去
      cloneObj[key] = cloneDeep(obj[key], hash);
    }
  }
  
  // 返回制作完成的复印件
  return cloneObj;
}
```

#### 总结

  - **问题**：循环引用会让简单的递归拷贝陷入**无限循环**，就像一个永远完不成工作的笨复印机。
  - **原理**：`cloneDeep` 的核心原理就像一个聪明的复印员，他带了一本**备忘录 (`WeakMap`)**。
  - **解决方案**：
    1.  每当要拷贝一个新对象时，先**查备忘录**，看是否拷贝过。
    2.  如果拷贝过，**直接返回备忘录里的记录**，打断循环。
    3.  如果没拷贝过，**立刻在备忘录里做好标记**，然后再去深入拷贝它的内容。

通过这种“**检查-记录-递归**”的模式，`cloneDeep` 巧妙地解决了循环引用的问题。

-----


## 四、区别

下面首先借助两张图，可以更加清晰看到浅拷贝与深拷贝的区别

 ![](https://static.vue-js.com/d9862c00-69b8-11eb-ab90-d9ae814b240d.png)

从上图发现，浅拷贝和深拷贝都创建出一个新的对象，但在复制对象属性的时候，行为就不一样

浅拷贝只复制属性指向某个对象的指针，而不复制对象本身，新旧对象还是共享同一块内存，修改对象属性会影响原对象

```js
// 浅拷贝
const obj1 = {
    name : 'init',
    arr : [1,[2,3],4],
};
const obj3=shallowClone(obj1) // 一个浅拷贝方法
obj3.name = "update";
obj3.arr[1] = [5,6,7] ; // 新旧对象还是共享同一块内存

console.log('obj1',obj1) // obj1 { name: 'init',  arr: [ 1, [ 5, 6, 7 ], 4 ] }
console.log('obj3',obj3) // obj3 { name: 'update', arr: [ 1, [ 5, 6, 7 ], 4 ] }
```

但深拷贝会另外创造一个一模一样的对象，新对象跟原对象不共享内存，修改新对象不会改到原对象

```js
// 深拷贝
const obj1 = {
    name : 'init',
    arr : [1,[2,3],4],
};
const obj4=deepClone(obj1) // 一个深拷贝方法
obj4.name = "update";
obj4.arr[1] = [5,6,7] ; // 新对象跟原对象不共享内存

console.log('obj1',obj1) // obj1 { name: 'init', arr: [ 1, [ 2, 3 ], 4 ] }
console.log('obj4',obj4) // obj4 { name: 'update', arr: [ 1, [ 5, 6, 7 ], 4 ] }
```



### 小结

前提为拷贝类型为引用类型的情况下：

- 浅拷贝是拷贝一层，属性为对象时，浅拷贝是复制，两个对象指向同一个地址

- 深拷贝是递归拷贝深层次，属性为对象时，深拷贝是新开栈，两个对象指向不同的地址
