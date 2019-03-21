# 命令模式

#### 用途

​	命令模式是最简单和优雅的模式之一，命令模式中的命令指的是一个执行某些特定事情的指令。

​	命令模式最常见的应用场景是：有时候需要向某些对象发送请求，但是并不知道请求的接收者是谁，也不知道被请求的操作是什么。此时希望用一种松耦合的方式来设计程序，使得请求发送者和请求接收者能够消除彼此之间的耦合关系。



#### 命令模式的例子-菜单程序



###### 闭包实现命令模式

```js
var setCommand = function(button, func) {
    button.onclick = function() {
		func();
	}
};

var MenuBar = {
    refresh: function() {
        console.log('刷新菜单界面');
    }
};

var RefreshMenuBarCommand = function( receicer ) {
    return function() {
		receiver.refresh();
    }
};

var refreshMenuBarCommand = RefreshMenuBarCommand( MenuBar );
setCommand( button1, refreshMenuBarCommand);
```

当然，如果想要更明确地表达当前正在使用命令模式，或者除了执行命令之外，将来有可能还要提供撤销命令等操作。那我们最好还是把执行函数改为调用execute方法：

```js
var RefreshMenuBarCommand = function( receiver ) {
    return {
        execute: function() {
            receiver.refresh();
        }
    }
};

var setCommand = function( button, command ) {
    button.onclick = function() {
        command.execute();
    }
};

var refreshhMenuBarCommand = RefreshMenuBarCommand( MenuBar );
setCommand( button1, refreshMenuBarCommand );
```



#### 撤销命令

命令模式的作用不仅是封装运算块，而且可以很方便地给命令对象增加撤销操作。就像订餐时客人可以通过电话来取消订单一样。

我们利用 [策略模式](Strategy.md) 中的Animate类来编写一个动画，这个动画的表现是让页面上的小球移动到水平方向的某个位置。现在页面有一个input文本框和一个button按钮，文本框中可以输入一些数字，表示小球移动后的水平位置，小球在用户点击按钮后立刻开始移动。

```html
<body>
    <div id="ball" style="position: absolute;background: #000; width:50px; height: 50px">
    </div>
    输入小球移动后的位置：<input id="pos" />
    <button id="moveBtn">
        开始移动
    </button>
    <button id="cancelBtn">
        cancel
    </button>
    
    <script>
    	var ball = document.getElementById('ball');
        var pos = document.getElementById('pos');
        var moveBtn = document.getElementById('moveBtn');
        var cancelBtn = document.getElementById('cancelBtn');
        
        var moveCommand = function( receiver, pos) {
			this.receiver = receiver;
            this.pos = pos;
            this.oldPos = null;
        };
        
        MoveCommand.prototype.execute = function() {
            this.receiver.start('left', this.pos, 1000, ' strongEaseOut');
            this.oldPos = this.receiver.dom.getBoundingClientRect()[this.receiver.propertyName]; // 记录小球开始移动前的位置
        };
        
        MoveCommand.prototype.undo = function() {
			// 回到小球移动前记录的位置
            this.receiver.start('left', this.oldPos, 1000, 'strongEaseOut');
        };
        
        var moveCommand;
        
        moveBtn.onclick = function() {
|			var animate = new Animate(ball);
            moveCommand = new MoverCommand( animate, pos.value);
            moveCommand.execute();
        };
        
        cancelBtn.onclick = function() {
            moveCommand.undo(); // 撤销命令
        }
    </script>
</body>
```

现在通过命令模式轻松地实现了撤销功能。如果用普通的方法调用来实现，也许需要每次都手工记录小球的运动轨迹，才能让它还原到之前的位置。而命令模式中小球的原始位置在小球开始移动前已经作为command对象的属性被保存起来，所以只需要再提供一个undo方法，并且再undo方法中让小球回到刚刚记录的原始位置就可以了。

#### 撤销和重做

很多时候，我们需要撤销一系列的命令。比如再一个围棋程序中，现在已经下了10步棋，我们需要一次性悔棋到第五步。再这之前，我们可以把所以执行过的下棋命令都存储再一个历史列表中，然后倒序循环来依次执行这些命令的undo操作，直到循环执行到第五个命令位置。

命令模式也可以用来实现播放录像功能。我们把用户在键盘的输入都封装成命令，执行过的命令将被存放到堆栈中。播放录像的时候只需要从头开始以此执行这些命令即可，代码如下

```html
<html>
    <body>
        <button id="replay">
            播放录像
        </button>
    </body>
    
    <script>
        var Ryu = {
            attack: function() {
                console.log('攻击');
            },
            defense: function() {
                console.log('防御');
            },
            jump: function() {
				console.log('跳跃');
            },
            crouch: function() {
                console.log('蹲下');
            }
        };
        
        var makeCommand = function( receiver, state ) { // 创建命令
            return function() {
                receiver[ state ]();
            }
        };
        
        var commands = {
            "119": "jump", 		// W
            "115": "crouch",	// S
            "97": "defense",	// A
            "100": "attack"		// D
        };
        
        var commandStack = []; // 保存命令的堆栈
        
        document.onkeypress = function( ev ) {
            var keyCode = ev.keyCode,
                command = makeCommand( Ryu, commands[keyCode] );
            
            if (command) {
                command(); // 执行命令
                commandStack.push(command); // 将刚刚执行过的命令保存进堆栈
            }
        };
        
        document.getElementById('replay').onclick = function() { // 点击播放录像
            var command;
            while( command = commandStack.shift() ) { // 从堆栈里一次取出命令并执行
                command();
            }
        };
    </script>
</html>
```



