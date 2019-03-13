# Vue面试题

1. 谈谈你对MVVM开发模式的理解

   ```
   MVVM分为Model、View、ViewModel三者.
   
   - Model: 代表数据模型，数据和业务逻辑都在Model层中定义；
   - View: 代表UI视图，负责数据的展示
   - ViewModel: 负责监听Model中数据的改变并控制视图的更新，处理用户交互操作；
   
   Model和View并无直接关联，而是通过ViewModel来进行联系的，Model和ViewModel之间有着双向数据绑定的联系。
   因此当Model中的数据改变时会触发View层的刷新，View中由于用户交互操作而改变的数据也会在Model中同步。
   
   这种模式实现了Model和View的数据自动同步，因此开发者只需要专注对数据的维护操作即可，而不需要自己操作dom。
   ```

   

2. Vue有哪些指令？

   ```
       v-model 双向绑定
       
       v-for 循环
       
       v-show 显示与隐藏（相当于css的overflow:hidden,个人理解）
       
       v-if 显示与隐藏（相当于css的display:none，个人理解）
       
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

3. 