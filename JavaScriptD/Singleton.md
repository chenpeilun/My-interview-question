# 单例模式

定义：保证一个类仅有一个实例，并提供一个访问它的全局访问点。

#### 1. 标准的单例模式

```js
var Singleton = function( name ) {
    this.name = name;
};
Singleton.prototype.getName = function() {
    alert( this.name );
};
Singleton.getInstance = (function() {
    var instance = null;
    return function( name ) {
        if (!instance) {
            instance = new Singleton( name );
        }
        return instance;
    }
})();
// 测试
var a = Singleton.getInstance('sven1');
var b = Singleton.getInstance('sven2');
alert(a === b); // true

```

通过`Singleton.getInstance`来获取Singleton类的唯一对象，这样子会增加了这个类的“不透明性”

下面将一步步编写出更好的单例模式

#### 2. 透明的单例模式

```js
var CreateDiv = (function() {
    var instance;
    
    
    var CreateDiv = function( html ) {
        if ( instance ) {
            return instance;
        }
        this.html = html;
        this.init();
        return instance = this;
    };
    
    CreateDiv.prototype.init = function() {
		var div = document.createElement( 'div' );
        div.innerHTML = this.html;
        document.body.appendChild(div);
    };
    return CreateDiv;
})();

// 测试
var a = new CreateDiv( 'sven1' );
var b = new CreateDiv( 'sven2' );

alert( a === b ); // true
```

为了把instance封装起来，我们使用了自执行的匿名函数和闭包，并且让这个匿名函数返回真正的Singleton构造方法，这样增加了一些程序的复杂读，阅读起来不是很舒服

如果哪天我们要让这个类从单例类变成一个普通的可以产生多个实例的类，那我们必须修改`CreateDiv`构造函数，但是这样的修改会给我们带来不必要的麻烦。

所以我们可以用下面的来解决上面这个问题。

#### 3. 代理实现单例模式

```js
var CreateDiv = function( html ) {
    this.html = html;
    this.init();
};
CreateDiv.prototype.init = function() {
    var div = doucment.getElementById('div');
    div.innerHTML = this.html;
    document.body.appendChild(div);
};

// 接下来引入代理类proxySingletonCreateDiv：
var ProxySingletonCreateDiv = (function() {
    var instance;
    return function( html ){
        if ( !instance ) {
            instance = new CreateDiv( html );
        }
        return instance;
    }
})();

// 测试
var a = new ProxySingletonCreateDiv('sven1');
var b = new ProxySingletonCreateDiv('sven2');
alert(a === b); // true
```

把负责管理单例的逻辑移到了代理类`proxySingletonCreateDiv`中。这样依赖，`CreateDiv`就变成了一个普通的类，它跟`proxySingletonCreateDiv`组合起来可以达到单例模式的效果。

#### 4. 惰性单例

惰性单例是指在需要的时候才创建对象实例，也是单例模式的重点。

假设我们要点击一个按钮的时候，会出现一个浮窗，但是这个浮窗在这个页面是唯一的，不可能同时存在两个浮窗的情况。



```js
// 将管理单例的逻辑从原来的代码中抽离出来，将这些逻辑封装在getSingle函数内部，
// 创建对象的方法fn被当成参数动态传入getSingle函数：
var getSingle = function( fn ) {
    var result;
    return function() {
        return result || ( result = fn.apply(this, arguments));
    }
}
// 接下来将用于创建登陆浮窗的方法用参数fn的形式传入getSingle，我们不仅可以传入
// createLoginLayer，还能传入createScript、createIframe、createXhr等。
// 之后再让getSingle返回一个新的函数，并且用一个变量result来保存fn的计算结果。
// result变量因为身在闭包中，永远不会被摧毁。
// 在将来的请求中，如果result已经被赋值，那么它将返回这个值。
var createLoginLayer = function() {
    var div = document.createElement('div');
    div.innerHTML = '浮窗';
    div.style.display = 'none';
    document.body.appendChild(div);
    return div;
};

var createSingleLoginLayer = getSingle( createLoginLayer );
docuemnt.getElementById('loginBtn').onclick = function() {
    var loginLayer = createSingleLoginLayer();
    loginLayer.style.display = 'block';
}
```

创建唯一的`iframe`用于动态加载第三方页面：

```js
var createSingleIframe = getSingle(function() {
    var iframe = docuemnt.createElement('iframe);
    document.body.appendChild(iframe);
    return iframe;
});
document.getElementById('loginBtn').onclick = function() {
    var loginLayer = createSingleIframe();
    loginLayer.src = 'http://baidu.com';
};
```

