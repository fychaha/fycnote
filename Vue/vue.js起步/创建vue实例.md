## Vue.js 起步

每个 Vue 应用都需要通过实例化 Vue 来实现。

语法格式如下：

`````javascript
var vm = new Vue({
    //构造参数
})
`````

构造参数

````vue
<div id="fyc">
    <h1>name : {{name}}</h1>//{{ }} <!-- 数据绑定,用于输出对象属性和函数返回值。 -->
    <h1>url : {{url}}</h1>
    <h1>{{port()}}</h1>
</div>
<script type="text/javascript">
    var vm = new Vue({
        el: '#fyc',//指定这个实例对哪个标签生效
        data: {
            name: "方秀波",
            url: "www.fyc.com",
            port: "2333"
        },//定义实例属性
        methods: {
            details: function() {//用于定义的函数，可以通过 return 来返回函数值
                return  this.name + " 假猪套天下第一";
            }
        }
    })
</script>
````

