前端面试题及其解答

---

[1. 假设现有一篇文章触及到一些敏感词汇,如 ["习近平","周永康","中共","6.4"] 等内容。如何在文章中发现这些敏感词，并将背景设置为红色或者改变字体颜色并标示出来？](#1)

[2. 下列输出值是什么？](#2)

1. 假设现有一篇文章触及到一些敏感词汇,如 ["习近平","周永康","中共","6.4"] 等内容。如何在文章中发现这些敏感词，并将背景设置为红色或者改变字体颜色并标示出来？

   ```javascript
   <style>
     .active{
     	background-color: #888;
       color:#fff;
   	}
   </style>

   <p id="text">Node.js®是一个基于Chrome V8 引擎的 JavaScript 运行时。 Node.js 使用高效、轻量级的事件驱动、非阻塞 I/O 模型。Node.js 之生态系统是目前最大的开源包管理		系统。</p>

   <script>
     	var text = document.getElementById('text').innerHTML;
       // 刷选条件
       var condition = ['Node.js','JavaScript','包管理系统'];
       var arr = text.split(condition[1]);

       // 1.根据条件筛选并将其替换为有类名的节点
       // 2.使用正则将匹配的内容替换成该节点
       // 3.将最后的内容设为根节点的innerHTML

       for(var k in condition){
         changeText(condition[k]);
       }

       function changeText(condition) {
         var span = createSpan(condition),
             regex = new RegExp(condition,'g');
         text = text.replace(regex,span);
         insert(text);
       }

       function createSpan(condition) {
         return '<span class="active">' + condition + '</span>';
       }

       function insert(content) {
         document.getElementById('text').innerHTML= content;
       }
   </script>
   ```

2. 下列输出值是什么？

   ```javascript
   # 1. b c 的值是多少
   var a = 100;

   function testResult(){    // 进入函数体内，创建当前执行上下文，创建阶段=>执行阶段
     var b = 2 * a;          // 创建阶段：创建变量对象，建立作用域，确定this指向
     var a = 200; 			  // 代码执行阶段，根据顺序，依次执行指令或数据操作（+/-/*）
     var c = a / 2; 
     console.log(b);
     console.log(c);
   }
   testResult();
   // 通过代码编译和执行顺序，改造内部执行流程
   // 1. 创建变量对象：建立arguments对象 => function函数声明提升 => 变量声明（其实设为undefined，如果与函数同名则跳过）
   // 2. 代码执行阶段
   actual function testResult(){
     var b = undefined;
     var a = undefined;
     var c = undefined;
     b = 2 * a; // b = 2 * undefined = NaN;
     a = 200;
     c = a / 2 ; // c = 200 / 2 = 100;
     console.log(b);// NaN
     console.log(c);// 100
    }

   #############################################

   # 2. z的值是多少
   var z = 10;
   function foo(){
     console.log(z);
   }
   (function(funArg){
     var z = 20;
     funArg();
   })(foo);
   // 同上对此函数体内流程进行改造
   actual (function(foo){
     var z = undefined;
     foo(); // 函数调用 => 创建执行上下文
   })(foo)// IIFE 立即调用函数表达式 => 创建执行上下文
   actual function foo(){
     console.log(z);// 内部执行上下文无法寻找到变量z，因此像作用域链下级（作用域链是栈解构，全局window执行上下文在最底层）寻找变量z
     // z = 10 ;
   }
   #############################################

   # 3. 依次输出的值是多少
   var data = [];
   for(var k = 0; k < 3; k++){
     // for 循环中无独立作用域
     data[k] = function(){
       console.log(k);
     };
   }
   data[0]();
   data[1]();
   data[2]();
   // data[k]都是相同的匿名函数
   //当函数调用时，创建执行上下文
   actual data[k] = function(){
     console.log(k);//函数体内部无变量K,向下追溯，在全局window执行上下文中找到变量k=3
     // k = 3
   }
   #############################################

   # 4. a,b,c输出分别是多少
   function fun(n,o){
     console.log(o);
     return{
      fun:function(m){
         return fun(m,n);
       }
     }; // return 结束
   }
   var a = fun(0); a.fun(1); a.fun(2); a.fun(3);
   var b = fun(0).fun(1).fun(2).fun(3);
   var c = fun(0).fun(1); c.fun(2); c.fun(3);
   // # 分析a: a = fun(0);
   actual function fun(0){
     console.log(o);// 未传参数，默认值为undefined，第一次输出 o = undefined
     return { // 返回含有fun属性的对象
       // n = 0
       fun:function(m){
         return fun(m,n);
       }
     }
   }
   // 得到a的函数
   a = fun(0) = {
     fun: function(m){
       return fun(m,0); // 返回fun函数，但含有默认参数o = 0
     }
   }
   // 接下来执行a函数的调用  a.fun(1); 函数体流程如下
     // m = 1 , n= 0
   a.fun(1) = function(1,0){
     console.log(o); // n = 1 , o = 0 第二次输出 o = 0
      return { // 返回含有fun属性的对象
        // n = 1 , o = 0
        fun:function(m){
          return fun(m,0);
        }
      }
   }
   // a的输出值为 undefined 0 0 0
   // a.fun(2) 和 a.fun(3)同理，输出值均为0，只是最终返回的函数所带参数不同

   // # 分析b: b = fun(0).fun(1).fun(2).fun(3);
   // 根据对a的分析，不难看出，当不断递归fun函数时，参数o从undefined，依次增加1
   // 输出值为 undefined 0 1 2

   // # 分析c: c = fun(0).fun(1);
   // 从分析b可以直接写出c的函数体内实际流程,fun(0)执行时第一次输出undefined
   c = fun(0).fun(1) = function(1,0){
     console.log(o); // n = 1 , o = 0 第二次输出 o = 0
      return { // 返回含有fun属性的对象
        // n = 1 , o = 0
        fun:function(m){
          return fun(m,1);
        }
      }
   }
   // 可以看出c起始就携带了参数（1，0）n = 1, o = 0
   c.fun(2) = function(2,1){
     console.log(o);// o = 1; 第三次输出 1
     return ...
   }
   //同理c.fun(3) 输出值也为 1
   ```