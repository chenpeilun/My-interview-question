# JavaScript中的深拷贝与浅拷贝

讲到深拷贝和浅拷贝前，先来说说什么是基本类型和引用类型

#### 基本类型和引用类型

ECMAScript变量包含两种不同数据类型的值： **基本类型值**和**引用类型值**

- **基本类型值**： 是一种保存在 **栈内存** 中的简单数据段，这个值完全保存在内存中的一个位置

  ```
  基本类型： Boolean、Null、undefined、Number、string、Symbol
  ```

  

- **引用类型值**：是一种保存在 **堆内存** 中的对象，变量中保存的实际上知识一个指针，这个指针指向内存中的保存改对象的位置。

  ```
  引用类型： Object、Function、Array
  ```

  

然而**深拷贝与浅拷贝的概念只存在于引用类型**

#### **Array**支持的深拷贝函数

```javascript
var arr1 = [1, 2]， arr2 = arr1.slice()；
console.log(arr1)；// [1, 2]
console.log(arr2)；// [1, 2]
arr2[1] = 3; // 修改arr2
console.log(arr1); // [1, 2]
console.log(arr2); // [1, 3]
```

可以看出arr2的修改并没有影响到arr1，这样也就实现了深拷贝，但是看看二维数组会不会也是这样？

继续将arr1改成二维数组

```javascript
var arr1 = [1,2,[3, 4]]， arr2 = arr1.slice()；
console.log(arr1)；// [1, 2,[3, 4]]
console.log(arr2)；// [1, 2,[3, 4]]
arr2[2][1] = 5; // 修改arr2
console.log(arr1); // [1, 2,[3, 5]]
console.log(arr2); // [1, 3,[3, 5]]
```

可以看出arr2的修改会影响到arr1，所以**slice()**只能实现一维数组的深拷贝

具备同等特性的还有： **concat**、**Array.form()**

#### Object支持的深拷贝函数

###### Object.assign()

Object.assign()只能实现一维对象的深拷贝

######  JSON.parse(JSON.stringify(obj))

*JSON.parse(JSON.stringify(obj)) *是可以实现深拷贝的，但是在MDN文档中写得很清楚

> > undefined、任意的函数以及 symbol 值，在序列化过程中会被忽略（出现在非数组对象的属性值中时）或者被转换成 null（出现在数组中时）。

所以 *JSON.parse(JSON.stringify(obj)) *的使用是有局限性的，不能深拷贝含有undefined、function、symbol值的对象，但是这个简单粗暴，已经满足90%的使用场景



#### 递归实现深拷贝

可以看见，经过发现，JS提供的自有方法并不能彻底解决Array、Object的深拷贝问题，所以想到了一种方法：递归

```javascript
function deepCopy() {
    // 创建一个新对象
    let result = {}
    let keys = Object.keys(obj),
        key = null,
        temp = null;
    for (let i = 0, l = keys.length; i < l ; i++) {
        key = keys[i];
        temp = obj[key];
        // 如果字段的值也是一个对象则进行递归操作
        if (temp && typeof temp === 'object') {
            result[key] = deepCopy(temp);
        } else {
            // 否则直接赋值给新对象
            result[key] = temp;
        }
    }
    return result;
}
```

递归完美解决了前面遗留的所有问题，其实第三方库中： **jquery的$.extend**和**lodash的_.cloneDeep**来解决深拷贝。上面对对象的深拷贝对Array也同样适用，因为Array也是特殊的Object。

#### 循环引用拷贝

```javascript
var obj1 = {
    x: 1,
    y: 2
};
obj1.z = obj1;
var obj2 = deepCopy(obj1);
```

在此时如果调用刚才的deepCopy函数，会陷入一个循环的递归的过程，从而导致爆栈。这个问题在jquery的$.extend中也存在。

其实解决这个问题只需要添加一个判断对象是否是引用了这个对象或这个对象的任意父级即可

```javascript
function deepCopy(obj, parent = null) {
    let result = {};
    let keys = Object.keys(obj),
        key = null,
        temp = null,
        _parent = parent;
    // 该字段有父级则需要追溯该字段的父级
    while (_parent) {
        // 如果该字段引用了它的父级则为循环引用
        if (_parent.originalParent === obj) {
            // 循环引用直接返回同级的新对象
            return _parent.currentParent;
        }
        _parent = _parent.parent;
    }
    for (let i = 0; i < keys.length; i++) {
        key = keys[i];
        temp = obj[key];
        // 如果该字段的值是一个对象
        if (temp && typeof temp === 'object') {
            // 递归执行深拷贝，将同级的待拷贝对象与新对象传递给parent方便追溯循环引用
            result[key] = deepCopy(temp, {
                originalParent: obj,
                currentParent: result,
                parent: parent
            });
        } else {
            result[key] = temp;
        }
    }
    return result;
}
```

这样已经完成一个支持循环引用的深拷贝函数。lodash的_.cloneDeep也可以实现