# Vue面试题

> 来源 https://github.com/chenpeilun/My-interview-question.git

1. #### 谈谈你对MVVM开发模式的理解

   ```
   MVVM分为Model、View、ViewModel三者.
   
   - Model: 代表数据模型，数据和业务逻辑都在Model层中定义；
   - View: 代表UI视图，负责数据的展示
   - ViewModel: 负责监听Model中数据的改变并控制视图的更新，处理用户交互操作；
   
   Model和View并无直接关联，而是通过ViewModel来进行联系的，Model和ViewModel之间有着双向数据绑定的联系。
   因此当Model中的数据改变时会触发View层的刷新，View中由于用户交互操作而改变的数据也会在Model中同步。
   
   这种模式实现了Model和View的数据自动同步，因此开发者只需要专注对数据的维护操作即可，而不需要自己操作dom。
   ```

   

2. #### Vue有哪些指令？

   ```
       v-model 双向绑定
       
       v-for 循环
       
       v-show 显示与隐藏（相当于css的display:block和display:none，个人理解）
       
       v-if 显示与隐藏（控制dom节点的出现或消失）
       
       v-else 必须与v-if连用
       
       v-else-if 必须与v-if连用
       
       v-bind 绑定数据
       
       v-on 事件绑定
       
       v-text 读取文本不能读取html标签
       
       v-html 能读取html标签
       
       v-style 样式
       
       v-once 进入页面时 只渲染一次 不再进行渲染
       
       v-cloak 防止闪烁
       
       v-pre 把标签内部的元素原为输出
       
   ```

3. #### v-if 和 v-show有什么区别

   ```
   v-show仅仅控制元素的显示方式，将display属性在block和none来回切换；
   
   v-if会控制这个DOM节点的存在与否。
   
   当我们需要经常切换某个元素的显示/隐藏时，使用v-show会更加节省性能上的开销；当只需要一次显示或隐藏时，使用v-if更加合理
   ```

   

4. #### 简述Vue的响应式原理

   ```
   当一个Vue实例创建时，vue会遍历data选项的属性，用Object.defineProperty将他们转为getter/setter并且在内部追踪相关依赖，在属性被访问和修改时通知变化。
   每个组件实例都有相应的watcher程序实例，它会在组件渲染的过程中把属性记录为依赖，之后当依赖项的setter被调用时，会通知watcher重新计算，从而致使它管理的组件得以更新
   ```

   ![img](https://mmbiz.qpic.cn/mmbiz_png/aVp1YC8UV0eopYpFgOLTRkR1w6ibZaCTqjFguS3KRUFeWO6GialKzqJwh2Lklibdogq7icwu1H2yQY0CbjrPlsa64Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

5. #### Vue中如何在组件内部实现一个双向数据绑定？

假设有一个输入框组件，用户输入时，同步父组件页面中的数据。

具体思路：父组件通过props传值给子组件，子组件通过 `$emit` 来通知父组件修改相应的props值，具体实现如下：

```vue
import Vue from 'vue'

const component = {
          props: [
            'value'
          ],
          template: `
            <div>
              <input type="text" @input="handleInput" :value="value">
    		</div>
  			`,
  data () {   
    return {
    }
  },
  methods: {
    handleInput (e) {
		this.$emit('input', e.target.value)
  	}
  }
}

new Vue({
  components: {
    CompOne : component
  },
  el: '#root',
  template: `
    <div>
      <comp-one :value1="value" @input="value = arguments[0]"></comp-one>
    </div>
  `,
  data () {
    return {
      value: '123'
    }
  }
})
```

6. #### Vue中如何监控某个属性值的变化？

   比如现在需要监控data中， `obj.a` 的变化。Vue中监控对象属性的变化你可以这样：

   ```vue
   watch: {
       obj: {
           handler(newValue, oldValue) {
   			console.log('obj changed)
   		}，
   		deep: true // 这里为true会对obj对象进行深层遍历，所以这里可以不要写
       }
   }
   ```

   还有一种方法，可以通过computed 来实现，只需要

   ```
   computed: {
       al() {
           return this.obj.a
       }
   }
   ```

   利用计算属性的特性来实现，当依赖改变时，便会重新计算一个新值。

   

7. #### Vue中给data中的对象属性添加一个新的属性时会发生什么，如何解决？

   示例：

   ```vue
   <template>
     
   	<div>
       	<ul>
         		<li v-for="value in obj" :key="value">
           		{{value}}
        		</li>
           </ul>
       <button @click="addObjB">添加obj.b</button>
     	</div>
   
   </template>
   <script>
   export default {
     data () {
       return {
         obj: {
           a: 'obj.a'
         }
       }
     },
     methods: {
       addObjB () {
         
   		this.obj.b = 'obj.b'
         	console.log(this.obj)
       }
     }
   }
   </script>
   <style></style>
   ```

   点击button会发现， `obj.b` 已经成功添加，但是视图并未刷新：

   ![img](https://mmbiz.qpic.cn/mmbiz_png/aVp1YC8UV0eopYpFgOLTRkR1w6ibZaCTqWZYJMl8DaILcvyTCeIx7gQ8u1200z6iaibhDXLUrC0ySVReQU4ZjI8rw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

   ![img](https://mmbiz.qpic.cn/mmbiz_png/aVp1YC8UV0eopYpFgOLTRkR1w6ibZaCTqvno4Rib5jx8eTaia0aWv1pAccEWIEuZHqgbzHtcBQ0o8duNVOLjMa7qQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

   原因在于在Vue实例创建时， `obj.b` 并未声明，因此就没有被Vue转换为响应式的属性，自然就不会触发视图的更新，这时就需要使用Vue的全局api—— `$set()`：

   ```vue
   addObjB () {     
     // this.obj.b = 'obj.b'
   
     this.$set(this.obj, 'b', 'obj.b')
     console.log(this.obj)
   }
   ```

   `$set()` 方法相当于手动的去把 `obj.b` 处理成一个响应式的属性，此时视图也会跟着改变了：

8. #### delete和Vue.delete删除数组的区别

   delete只是被删除的元素变成了empty/undefined其他的元素的键值还是不变

   Vue.delete直接删除了数据改变了数组的键值。

   ```
    
   var a=[1,2,3,4]
       
   var b=[1,2,3,4]
   delete a[1]
   console.log(a)
       
   this.$delete(b,1)
   console.log(b)
   ```

![img](https://mmbiz.qpic.cn/mmbiz_png/aVp1YC8UV0eopYpFgOLTRkR1w6ibZaCTqjMuDlmqHePL2N3zgc2UicYobuG60QcEFdAfUZUuVICajN9IB0ibQ3rqA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)![img](https://mmbiz.qpic.cn/mmbiz_png/aVp1YC8UV0eopYpFgOLTRkR1w6ibZaCTqxPHafNBqkq8I4uZ40P1iaXiaeWpaKAGIVoKAibibKDs4u49LPibfrANkrXg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

#### 