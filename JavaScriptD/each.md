# 迭代器模式

迭代器模式是指提供一种方法顺序访问一个聚合对象中的各个元素，而又不需要暴露该对象的内部表示。

迭代器模式可以把迭代的过程从业务逻辑中分离出来，在使用迭代器模式之后，即使不关心对象的内部构造，也可以按顺序访问其中的每个元素。



#### 实现自己的迭代器

```js
// each接受2个参数
// arr: 被循环的数组
// callback: 循环中的每一步后将被触发的回调函数.
var each = function(arr, callback) {
    for (var i = 0, l = arr.length; i < l; i++) {
		callback.call( arr[i], i, arr[i]); // 把下标和元素当作参数传递给callback函数		
	}
};
each( [1, 2, 3], function(i, n) {
    alert([i, n]);
});
```



#### 内部迭代器

上面的each迭代器就是一个内部迭代器。也就是在each函数的内部已经定义好迭代规则，它完全接手整个迭代过程，外部只需要一次初始调用。

虽然内部迭代器在使用的时候非常方便，外界不用关心迭代器内部的实现，跟迭代器的交互也仅仅是一次初始调用，但是这也刚好是内部迭代器的缺点。由于内部迭代器的迭代规则已经被提前规定，上面的each函数就无法同时迭代两个数组了。

如果要判断2个数组里的元素的值是否相等,在不改变each函数的前提上，我们只能从callback中进行修改了。

```js
var compase = function(arr1, arr2) {
    if (arr1.length !== arr2.length) {
        throw new Error('arr1和arr2不想等');
    }
    each(arr1, function(i, n) {
        if (n !== arr2[i]) {
            throw new Error('arr1和arr2不相等');
        }
    });
    alert('arr1和arr2相等');
}

compase( [1, 2, 3], [1, 2, 4])； // throw new Error('arr1和arr2不相等)； 
```



#### 外部迭代器

外部迭代器必须显式地请求迭代下一个元素

```js
var Iterator = function(obj) {
    var current = 0;
    
    var next = function() {
        current += 1;
    };
    
    var isDone = function() {
		return current >= obj.length;
    };
    
    var getCurrItem = function() {
        return obj[current];
    };
    
    return {
		next: next,
        isDone: isDone,
        getCurrItem: getCurrItem
    }
}

var compare = function(iterator1, iterator2) {
    while( !iterator1.isDone() && !iterator2.isDone()) {
        if (iterator1.getCurrItem() !== iterator2.getCurrItem()) {
            throw new Error('iterator1 和 iterator2 不相等');
        }
        iterator1.next();
    	iterator2.next();
    }
    alert( 'iterator1 和 iterator2 相等');
    
}

var iterator1 = Iterator([1, 2, 3]);
var iterator2 = Iterator([1, 2, 3]);
compare(iterator1, iterator2); // 输出： iterator1 和iterator2 相等

```



#### 倒序迭代器

```js
var reverseEach = function( arr, callback) {
    for (var l = arr.length - 1; l >= 0; l--) {
        callback(l, arr[l]);
    }
};

reverseEach([0, 1, 2], function(i, n) {
    console.log(n); // 分别输出: 2, 1, 0
});
```



