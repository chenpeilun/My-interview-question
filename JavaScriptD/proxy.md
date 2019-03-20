# 代理模式

代理模式是为了一个对象提供一个代用品或占位符，以便控制对它的访问。



代理模式的关键是，当客户不方便直接访问一个对象或者不满足需要的时候，提供一个替身对象来控制对这个对象的访问，客户实际上访问的是替身对象。替身对象对请求做出一些处理之后，再把请求转交给本体对象。

#### 1. 保护代理

保护代理就是说代理类能过滤掉一些请求。不需要本体去过滤

比如小明想要送花给A，但是是通过代理B去送。这时B就可以过滤一下小明的资格，是不是可以给A送花，不然就拒绝，A就不需要过滤。

保护代理用于控制不同权限的对象对目标对象的访问，但在JavaScript并不容易实现保护代理，因为我们无法判断谁访问了某个对象。

#### 2. 虚拟代理

​	虚拟代理把一些开销很大的对象，延迟到真正需要它的时候才去创建.

###### 虚拟代理实现图片预加载

​	在web开发中,图片预加载是一种常用的技术,如果直接给某个`img`标签节点设置`src`属性,由于图片过大或者网络不佳,图片的位置往往有段时间会是空白.

​	常见的解决方法是先用一张loading图片占位,然后用异步的方式加载图片,等图片加载好了再把它填充到`img`节点里,这种场景就适合使用虚拟代理.

```js
// 创建一个普通的本体对象,这个对象负责往页面中创建一个img标签,并且提供一个对外的setSrc接口
var myImage = (function() {
    var imgNode = document.createElement('img');
    document.body.appendChild(imgNode);
    
    return {
        setSrc: function( src ) { // 外界调用这个接口,便可以给该img标签设置src属性
            imgNode.src = src;
        }
    }
})();

// 引入代理对象proxyImage,通过这个代理对象,在图片被真正加载号前,页面中将出现一张占位的菊花图loading.gif,来提示用户图片正在加载.
var proxyImage = (function() {
    var img = new Image;
    img.onload = function() {
        myImage.setSrc( this.src );
    }
    return {
        setSrc: function( src ) {
            myImage.setSrc('file:// /C:/users/xxx.gif');
            img.src= src;
        }
    }
})();

proxyImage.setSrc('http://imgcache.qq.com/m.jpg');
```

现在我们通过`proxyImage`间接地访问`MyImage.proxyImage`控制了客户对`MyImage`的访问.



###### 单一职责原则

​	单一职责原则指的是,就一个类(通常也包括对象和函数)而言,应该仅有一个引起它变化的原因.如果一个对象承担了多项职责,就意味着这个对象将变得巨大,引起它变化的原因可能会有多个.面向对象设计鼓励将行为分布到细粒度的对象之中,如果一个对象承担的职责过多,等于把这些职责耦合到了一起,这种耦合会导致脆弱和低内聚的设计.当变化发生时,设计可能会遭到意外的破坏.

​	为什么要把设置`src`和图片预加载分成两个对象来实现呢,就是要到将来,如果网速很好的情况,不需要到图片预加载了,这个代码可以去掉了,我们不需要去改动`MyImage`对象的代码,而只需要删除`proxyImage`对象就可以了.

​	

###### 虚拟代理合并HTTP请求

```js
var symcjrpmpisFile = function( id ) {
    console.log('开始同步文件, id为:' + id);
};

// 收集一段时间之内的请求,最后一次性发生给服务器.
// 比如我们等到2秒之后才把这2秒内需要同步的文件ID打包发送给服务器.
var proxySynchronousFile = ( function() {
    var cache = [], // 保存一段时间内需要同步的ID
        timer; // 定时器
    
    return function(id) {
        cache.push(id);
        if (timer) { // 保证不会覆盖已经启动的定时器
            return;
        }
        
        timer = setTimeout(function() {
			synchronousFile( cache.join(',')); // 2秒后向本体发生需要同步的ID集合
            clearTimeout( timer ); // 清空定时器
            timer = null;
            cache.length = 0; // 清空ID集合
        }, 2000);
    }
})();

var checkbox = document.getElementByTagName('input');
for (var i = 0, c; c = checkbox[i++]) {
    c.onclick = function() {
        if ( this.checked === true) {
            proxySynchronousFile( this.id );
        }
    }
};
```



###### 虚拟代理在惰性加载中的应用

如果我们需要点击一个按键的时候才去页面加载一个`js`文件,也就是说用户有需要的时候才去加载.而且要保证,用户在重复按键的时候,这个`js`文件只被加载一次.

```js
var miniConsole = ( function() {
    var cache = [];
    var handler = function(ev) {
        if (ev.keyCode === 113) {
            var script = document.createElement('script');
            script.onload = function() {
                for (var i = 0, fn; fn = cache[i++];) {
                    fn();
                }
            };
            script.src = 'xxx.js';
            document.getElementByTagName('head')[0].appendChild(script);
            // 只加载一次xxx.js
            document.body.removeEventListener('keydown', handler);
        }
    };
    
    document.body.addEventListener('keydown', handler, false);
    
    return {
        log: function() {
			var args = arguments;
            cache.push( function() {
                return miniConsole.log.apply(miniConsole, args);
            })
        }
    }
})();

miniConsole.log(11);

// xxx.js
miniConsole = {
    log: function() {
        console.log(Array.prototype.join.call(arguments));
    }
}
```



#### 3. 缓存代理

```js
// 计算乘积
var mult = function() {
    var a = 1;
    for ( var i = 0, l = arguments.length; i < 1; 1++) {
		a = a * arguments[i];
	}
	return a;
};

// 计算加和
var plus = function() {
    var a = 0;
    for ( var i = 0, l = arguments.length; i < l; i++) {
        a = a + arguments[i];
    }
    return a;
};

// 创建缓存代理的工厂
var createProxyFactory = function( fn ) {
    var cache = {};
    return function() {
        var args = Array.prototype.join.call(arguments, ',');
        if ( args in cache ) {
            return cache[args];
        }
        return cache[args] = fn.apply(this, arguments);
    }
};

var proxyMult = createProxyFactory(mult),
   	proxyPlus = createProxyFactory(plus);

alert( proxyMult(1, 2, 3, 4)); // 输出24
alert( proxyMult(1, 2, 3, 4)); // 输出24
alert( proxyPlus(1, 2, 3, 4)); // 输出10
alert( proxyPlus(1, 2, 3, 4)); // 输出10
```

