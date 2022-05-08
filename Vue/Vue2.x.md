# Vue2.x

## 安装

---

1. 可以直接下载js后通过`<script>`标签引入，`Vue` 会被注册为一个全局变量。(建议在生产环境用`vue.min.js`)
2. 使用`CDN`
   - 对于制作原型或学习，你可以这样使用最新版本： `<script src="https://cdn.jsdelivr.net/npm/vue@2.6.14/dist/vue.js"></script>`
   - 对于生产环境，我们推荐链接到一个明确的版本号和构建文件，以避免新版本造成的不可预期的破坏：`<script src="https://cdn.jsdelivr.net/npm/vue@2.6.14"></script>`

## 初识Vue

---

1. 想让Vue工作，必须创建一个Vue实例，且要传入一个配置对象
2. root容器里的代码依然符合html规范，只不过混入了一些特殊的Vue语法
3. root容器里的代码被称为 Vue模版
4. 真实开发中只有一个Vue实例，并且会配合着组件一起使用
5. `{{{xx}}}`中的`xx`要写js表达式，且xx可以自动读取到data中的所有属性
6. 一旦data中的数据发生改变，那么模版中用到该数据的地方也会自动更新

**注意区分：**js表达式和js代码（语句）
- 表达式：一个表达式会产生一个值，可以放在任何一个需要值的地方
  1. a
  2. a+b
  3. demo(1)
  4. x==y?a:b
- js代码（语句）
  1. if(){}
  2. for(){}

Vue 模版语法有两大类：
1. 插值语法：
   - 功能：用于解析标签体内容
   - 写法：{{xxx}},xxx是js表达式，且可以直接读取到data中的所有属性
2. 指令语法：
   - 功能：用于解析标签（包括：标签属性、标签体内容、绑定事件等）
   - 举例：`v-bind:href="xxx"` 或 简写为 `:href="xxx"`,xxx同样要写js表达式
   - **备注：** Vue中有很多指令，且形式都是`:-???`，此处只拿`v-bind`举例

代码如下：

```csharp
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>初识Vue</title>
    <!-- 引入Vue -->
    <script type="text/javascript" src="../js/vue.js"></script>
</head>

<body>

    <!-- 准备好一个容器 -->
    <div id="root">
        <h1>插值语法</h1>
        <h3>Hello,{{name}},{{Date.now()}}</h3>

        <h1>指令语法</h1>
        <a v-bind:href="url.toUpperCase()">点我链接到{{blog.name1}}</a>
        <!-- v-bind： 简写 -->
        <a :href="url">点我链接到{{blog.name2}}</a>
    </div>

    <script type="text/javascript">
        Vue.config.productionTip = false //阻止vue在启动时生成生产提示

        // 创建Vue实例
        new Vue({
            // el:document.getElementById('root') 不常用，作为了解
            el: '#root', //el用于指定当前Vue实例为哪个容器服务，值通常为css选择器字符串

            data: { // data中用于存储数据，数据供el所指定的容器使用
                name: 'xx',
                url:"http://xxxerxes.github.io",
                // 层级结构
                blog:{
                    name1:"Blog1",
                    name2:"Blog2"
                }
            }
        })
    </script>
</body>

</html>
```