# 发布-订阅模式

发布-订阅模式又叫观察者模式，它定义对象间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都将得到通知。



#### 实现一个简单的发布-订阅模式

​	例子：小明看上一套房子，到了售楼处才被告知，该楼盘已经售罄了。小明离开前，把电话号码留在了售楼处。售楼MM答应他，新楼盘一推出就马上发信息通知小明。还有好几个人也是一样，所有我们就要模拟一下这个场景，设计一个程序，来进行通知。

```js
var salesOffices = {}; // 定义售楼处

salesOffices.clientList = []; // 缓存列表，存放订阅者的回调函数

salesOffices.listen = function( key, fn) {
    if ( !this.clientList[key] ) { // 如果还没有订阅过此类消息，给该类消息创建一个缓存列表
        this.clientList[key] = [];
    }
    this.clientList[key].push(fn); // 订阅的消息添加进消息缓存列表
};

salesOffices.trigger = function() { // 发布消息
    var key = Array.prototype.shift.call(arguments), // 取出消息类型
        fns = this.clientList[key]; // 取出该消息对应的回调函数集合
    
    if ( !fns || fns.length === 0 ) { // 如果没有订阅该消息，则返回
        return false;
    }
    
    for (var i = 0, fn; fn = fns[i++]; ) {
        fn.apply(this, arguments); // arguments是发布消息时附送的参数
    }
    
    salesOffices.listen( 'squareMeter88', function(price) { // 小明订阅88平方米房子的消息
        console.log('价格=' + price); // 输出： 2000000
    });
    salesOffices.listen('squareMeter110', function(price) { // 小红订阅110平方米房子的消息
        console.log('价格=' + price); // 输出：3000000
    })；
    
    salesOffices.trigger('squareMeter88', 2000000); // 发布88平方米房子的价格
    salesOffices.trigger('squareMeter110', 3000000); // 发布110平方米房子的价格
}
```



#### 全局的发布-订阅对象

我们上面写的发布-订阅模式代码还有些问题

- 我们给每个发布者对象都添加了listen和trigger方法，以及一个缓存列表`clientList`，这其实是一种资源浪费。
- 小明跟售楼处对象还是存在移动的耦合性，小明至少要指导售楼处对象的名字是`salesOffices`，才能顺利的订阅到事件。

其实在现实中，买房子未必要亲自取售楼处，我们只要把订阅的请求交给中介公司，而各大房产公司也只需要通过中介公司来发布房子信息。

同样在程序中，发布-订阅模式也可以用一个全局的Event对象来实现，订阅者不需要了解消息来自哪个发布者，发布者也不知道消息会推送给哪些订阅者，Event作为一个类似“中介者”的角色，把订阅者和发布者联系起来。

```js
var Event = (function() {
	var clientList = {},
        listen,
        trigger,
        remove;
    
    listen = function(key, fn) {
        if ( !clientList[key] ) {
            clientList[key] = [];
        }
        clientList[key].push(fn);
    };
    
    trigger = function() {
		var key = Array.prototype.shift.call(arguments),
            fns = clientList[key];
        if ( !fns || fns.length === 0) {
            return false;
        }
        for (var i = 0, fn; fn = fns[i++]; ) {
            fn.apply(this, arguments);
        }
    };
    
    remove = function( key, fn ) {
        var fns = clientList[key];
        if ( !fns ) {
            return false;
        }
        if ( !fn ) {
            fns && (fns.length = 0 );
        }else {
            for (var l = fns.length - 1; l >= 0; l--) {
                var _fn = fns[l];
                if ( _fn === fn) {
                   fns.splice( l, 1);
                }
            }
        }      
    };
    
    return {
           listen: listen,
           trigger: trigger,
           remove: remove
    }
})();

Event.listen('squareMeter88', function(price) { // 小明订阅消息
    console.log('价格=' + price); // 输出： ‘价格=2000000’
})

Event.trigger('squareMeter88', 2000000); // 售楼处发布消息
```

