# 策略模式

#### 1. 策略模式的定义

​	定义一系列的算法，把它们一个个封装起来，并且使它们可以互相替换。

#### 2. 使用策略模式计算奖金

​	很多公司的年终奖是根据员工的工资基数和年底绩效情况来发放的。例如，绩效为S的人年终奖有4倍工资，绩效为A的人年终奖有3倍工资，而绩效为B的人年终奖是2倍工资。

```js
// 在JavaScript语言中，函数也是对象，所以直接把strategy直接定义为函数
var strategies = {
    "S": function( salary ) {
        return salary * 4;
    },
    "A": function( salary ) {
        return salary * 3;
    },
    "B": function( salary ) {
        return salary * 2;
    }
};

var calculateBonus = function( level, salary ) {
    return strategies[level]( salary );
};

console.log( calculateBonus('S', 20000)); // 输出： 80000
console.log( calculateBonus('A', 10000)); // 输出： 30000
```

#### 3. 使用策略模式实现缓动动画

###### 让小球运动起来

​	分析实现这个程序的思路

- ​	动画开始时，小球所在的原始位置；

- ​	小球移动的目标位置；

- ​	动画开始时的准确时间点；

- ​	小球运动持续的事件。

  随后，用`setInterval`创建一个定时器，定时器每隔`19ms`循环一次。在定时器的每一帧里，我们会把动画已消耗的时间、小球的原始位置、小球母表位置和动画持续的总时间等信息传入欢动算法。该算法会通过这几个参数，计算出小球当前应该所在的位置。最后再更新该`div`对应的`css`属性，小球就能够顺利的运动起来。

  ```js
  xxx.js
  // Flash中的缓动算法
  var tween = {
      linear: function( t, b, c, d) {
          return c*t/d + b;
      },
      easeIn: function( t, b, c, d) {
          return c * ( t /= d ) * t + b;
      },
      strongEaseIn: function( t, b, c, d) {
          return c * ( t /= d ) * t * t * t * t + b;
      },
      strongEaseOut: function( t, b, c, d) {
          return c * ( ( t = t / d - 1 ) * t * t * t * t + 1 ) + b;
      },
      sineaseIn: function(t, b, c, d) {
          return c * ( t /= d ) * t * t + b;
      },
      sineaseOut: function(t, b, c, d) {
  		return c * ((t = t / d - 1) * t * t + 1) + b;
      }
  };
  // Animate类，Animate构造函数接受一个参数：即将运动起来的dom节点。
  var Animate = function(dom) {
      this.dom = dom;				// 进行运动的dom节点
      this.startTime = 0;			// 动画开始时间
      this.startPos = 0;			// 动画开始时，dom节点位置，即dom的初始位置
      this.endPos = 0;			// 动画结束时，dom节点位置，即dom的母表位置
      this.propertyName = null;	// dom节点需要被改变的css属性名
      this.easing = null;			// 缓动算法
      this.duration = null;		// 动画持续时间
  }
  
  // Animate.prototype.start方法负责启动这个动画
  // propertypeName: 要改变的CSS属性名，比如'left'、'top'，分别表示左右移动和上下移动。
  // endPos: 小球运动的目标位置。
  // duration: 动画持续时间。
  // easing: 缓动算法。
  Animate.prototype.start = function ( propertyName, endPos, duration, easing) {
      this.startTime = +new Date; // 动画启动时间
      this.startPos = this.dom.getBoundingClientRect()[propertyName]; // dom节点的初始位置
      this.propertyName = propertyName; // dom节点需要被改变的css属性名
      this.endPos = endPos; // dom节点目标位置
      this.duration = duration; // 动画持续事件
      this.easing = tween[easing]; // 缓动算法
      
      var self = this;
      var timeId = setInterval(function() { // 启动定时器，开始执行动画
          if (self.step() === false) { // 如果动画已结束，则清除定时器
              clearInterval( timeId );
          }
      }， 19)；
  }
  
  // Animate.prototype.step
  // 这个方法负责计算小球的当前位置和调用更新css属性值的方法Animate.prototype.update。
  Animate.protytype.step = function() {
      var t = +new Date; // 取得当前时间
      // 如果当前时间大于动画开始时间加上动画持续时间之和，说明动画已经结束
      // 此时要修正小球的位置。
      if (t >= this.startTime + this.duration) {
          this.update( this.endPos ); // 更新小球的CSS属性值
          return false;
      }
      var pos = this.easing( t - this.startTime, this.startPos, this.endPos - this.startPos, this.duration );
      // pos为小球当前的位置
      this.update(pos); // 更新小球的CSS属性值
  };
  
  // Animate.prototype.update方法
  // 负责更新小球CSS属性值
  Animate.prototype.update = function(pos) {
      this.dom.style[ this.propertyName ] = pos + 'px';
  };
  
  ```

  测试

  ```html
  <html>
      <head>
          
      </head>
      <body>
          <div style="position: absolute; background: blue" id="div">
              DIV
          </div>
          <script src="xxx.js"></script>
          <script>
          	var div = document.getElementById('div');
              var animate = new Animate(div);
              animate.start('left', 500, 1000, 'strongEaseOut');
              // animate.start('top', 1500, 500, 'strongEaseIn');
          </script>
      </body>
  </html>
  ```

  ###### 表单验证

  ```js
  var startegies = {
      isNonEmpty: function( value, errorMsg) {
          if (value === '') {
              return errorMsg;
          }
      },
      minLength: function( value, length, errorMsg) {
          if (value.length < length) {
              return errorMsg;
          }
      },
      isMobile: function( value, errorMsg ) {
          if ( !/(^1[3|5|8](0-9){9}$)/.test(value)) {
              return errorMsg;
          }
      }
  }
  
  var Validator = function() {
      this.cache = []; // 保存校验规则
  };
  
  Validator.prototype.add = function( dom, rule, errorMsg) {
  	var ary = rule.split(':'); // 把strategy和参数分开
      this.cache.push(function() { // 把校验的步骤用空函数包装起来，并且放入cache
          var strategy = ary.shift(); // 用户挑选的strategy
          ary.unshift(dom.value); // 把input的value添加进参数列表
          ary.push(errorMsg);	//把errorMsg添加进参数列表
          return strategies[strategy].apply( dom, ary);
      });
  };
  
  Validator.prototype.start = function() {
      for (var i = 0; validatorFunc; validatorFunc = this.cache[i++];) {
          var msg = validatorFunc(); // 开始校验，并取得校验后的返回信息
          if (msg) { // 如果有确切的返回值，说明校验没有通过
              return msg;
          }
      }
  }
  
  var validataFunc = function() {
      var validator = new Validator(); // 创建一个validator对象
      
      // 添加校验规则
      validator.add ( registerForm.username, 'isNonEmpty', '用户名不能为空');
      validator.add ( registerForm.password, 'minLength:6', '密码长度不能少于6位');
      validator.add ( registerForm.phoneNumber, 'isNonEmpty', '手机号码格式不正确');
      var errorMsg = validator.start(); //获得校验结果
      return errorMsg; // 返回校验结果
  }
  
  var registerForm = document.getElementById( 'registerForm' );
  registerForm.onsubmit = function() {
      var errorMsg = validataFunc();
      if (errorMsg) {
          alert(errorMsg);
          return false; //阻止表单提交
      }
  }
  ```

  但是这个表单验证只能实现单个验证，无法给某个文本输入框添加多种校验规则