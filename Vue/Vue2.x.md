# Vue2.x

## 安装

---

1. 可以直接下载js后通过`<script>`标签引入，`Vue` 会被注册为一个全局变量。(建议在生产环境用`vue.min.js`)
2. 使用`CDN`
   - 对于制作原型或学习，你可以这样使用最新版本： `<script src="https://cdn.jsdelivr.net/npm/vue@2.6.14/dist/vue.js"></script>`
   - 对于生产环境，我们推荐链接到一个明确的版本号和构建文件，以避免新版本造成的不可预期的破坏：`<script src="https://cdn.jsdelivr.net/npm/vue@2.6.14"></script>`

## 模版语法

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
  4. x===y?a:b
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

```html
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

## 数据绑定

---

Vue中有2种数据绑定的方式：
1. 单向绑定(`v-bind`):数据只能从data流向页面
2. 双向绑定(`v-model`):数据不仅能从data流向页面，还可以从页面流向data
   **Remark:**
   1. 双向绑定一般都应用在表单类元素上（如：`input`,`select`等)
   2. `v-model:value`可以简写为`v-model`

```html
<body>

    <!-- 准备好一个容器 -->
    <div id="root">
        单向数据绑定：<input type="text" v-bind:value="name"> <br>
        双向数据绑定：<input type="text" v-model:value="name">

        <br>
        
        <!-- 简写 -->
        单向数据绑定：<input type="text" :value="name"> <br>
        双向数据绑定：<input type="text" v-model="name">
    </div>

    <script type="text/javascript">
        Vue.config.productionTip = false //阻止vue在启动时生成生产提示

        // 创建Vue实例
        new Vue({
            el: '#root',

            data: {
                name: 'xx'
            }
        })
    </script>

</body>
```

## el与data的两种写法

---

el与data的两种写法:
1. el有2种写法
   1. `new Vue`时配置el属性
   2. 先创建Vue实例，随后通过`vm.$mount('#root')`指定el的值
2. data有2种写法
   1. 对象式
   2. 函数式 使用组件时必须使用函数式

**Tips:** 由Vue管理的函数，一定不要写箭头函数，一旦写了，this就不再是Vue实例了

```html
<script type="text/javascript">
    Vue.config.productionTip = false //阻止vue在启动时生成生产提示

    // 创建Vue实例
    const v = new Vue({
        // el: '#root', //第一种写法

        // data: { // data的第一种写法，对象式
        //     name: 'xx'
        // }

        data() {
            return {
                name: 'xx'
            }
        }
    })
    v.$mount('#root') // 第二种写法

</script>
```

## MVVM模型

---

M: 模型（Model）对应data中的数据  
V: 视图（View） 模版  
VM: 视图模型（ViewModel) Vue实例对象  

data中的所有属性，都能在vm上找到  
vm的所有属性以及Vue原型上的所有属性，在Vue模版中都能直接使用

```html
<div id="root">
    <h1>name: {{name}}</h1>
    <h1>schoolName: {{schoolName}}</h1>
</div>

<script type="text/javascript">
    Vue.config.productionTip = false

    // 创建Vue实例
    const vm = new Vue({
        el: '#root', 

        data: {
            name: 'xx',
            schoolName:'xxSchool'
        }
    })
</script>
```

## 数据代理

---

### defineProperty

```html
<script type="text/javascript">
    Vue.config.productionTip = false

    let number = 18
    let person = {
        name:'xx',
        sex:'male'
    }

    Object.defineProperty(person,'age',{
        // value:18,
        // enumerable:true, // 控制属性是否可枚举，默认值为false
        // writable:true, //控制属性是否可以被修改，默认值为false
        // configurable:true, //控制属性是否可以被删除，默认值是false

        get(){
            return number
        },

        set(value){
            number = value
        }
    })

    //console.log(Object.keys(person))
    console.log(person)
</script>
```

### 何为数据代理

数据代理：通过一个对象代理对另一个对象中属性的操作(读/写)

```html
<script type="text/javascript">
    let obj1 = {x:100}
    let obj2 = {y:200}

    Object.defineProperty(obj2,'x',{
        get(){
            return obj1.x
        },

        set(value){
            obj1.x = value
        }
    })
</script>
```

## 事件处理

---

事件的基本使用：
1. 使用`v-on:xx` 或 `@xx`绑定事件，其中xx是事件名
2. 事件的回调需要配置在methods对象中，最终会在vm上
3. methods中配置的函数，不要用箭头函数，否则this就不是vm了
4. methods中配置的函数，都是被Vue所管理的函数，this的指向是vm或组件实例对象
5. `@click="demo"` 和 `@click="demo($event)"`效果一致，但后者可以传参

```html
<body>
    <div id="root">
        <h2>Hello,{{name}}</h2>
        <button v-on:click="showInfo1">点我提示1</button>
        <!-- 简写 -->
        <button @click="showInfo2($event,xx)">点我提示2</button>
    </div>

    <script type="text/javascript">
        Vue.config.productionTip = false

        // 创建Vue实例
        const vm = new Vue({
            el: '#root',
            data: {
                name: 'xx'
            },
            methods:{
                showInfo1(event){
                    //console.log(event.target.innerText)
                    //console.log(this) //此处的this是vm
                    alert("hello")
                },
                showInfo2(event,name){
                    alert("hi")
                }
            }
        })
    </script>
</body>
```

### Vue中的事件修饰符

1. prevent:阻止默认事件（常用）
2. stop:阻止事件冒泡（常用）
3. once:事件只触发一次（常用）
4. capture:使用事件的捕获模式
5. self:只有event.target是当前操作的元素时才触发事件
6. passive:事件的默认行为立即执行，无需等待事件回调执行完毕

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>事件处理</title>
    <script type="text/javascript" src="../js/vue.js"></script>
    <style>
        *{
            margin-top: 20px;
        }
        .demo1{
            height: 50ox;
            background-color: skyblue;
        }
        .box1{
            padding: 5px;
            background-color: skyblue;
        }
        .box2{
            padding: 5px;
            background-color: orange;
        }
        .list{
            width: 200px;
            height: 200px;
            background-color: yellow;
            /* 滚动条 */
            overflow: auto;
        }
    </style>
</head>

<body>
    <div id="root">
        <!-- 阻止默认事件（常用） -->
        <a href="xxxerxes.github.io" @click.prevent="showInfo">点我提示信息</a>
        <!-- 阻止事件冒泡（常用） -->
        <div class="demo1" @click="showInfo">
            <button @click.stop="showInfo">点我提示信息</button>
        </div>
        <!-- 事件只触发一次（常用） -->
        <button @click.once="showInfo">点我提示信息</button>
        <!-- 使用事件的捕获模式 -->
        <div class="box1" @click.capture="showMsg(1)">
            div1
            <div class="box2" @click="showMsg(2)">
                div2
            </div>
        </div>
        <!-- 只有event.target是当前操作的元素时才触发事件 -->
        <div class="demo1" @click.self="showInfo">
            <button @click="showInfo">点我提示信息</button>
        </div>
        <!-- 事件的默认行为立即执行，无需等待事件回调执行完毕 -->
        <ul @wheel.passive="demo" class="list">
            <li>1</li>
            <li>2</li>
            <li>3</li>
            <li>4</li>
        </ul>
    </div>

    <script type="text/javascript">
        Vue.config.productionTip = false

        // 创建Vue实例
        const vm = new Vue({
            el: '#root',
            data: {
                name: 'xx'
            },
            methods:{
                showInfo(event){
                    //event.preventDefault()
                    alert("hello!")
                },
                showMsg(msg){
                    console.log(msg)
                },
                demo(){
                    console.log('@')
                }
            }
        })
    </script>
</body>

</html>
```

### 键盘事件

1. Vue中常用的按键别名：
- 回车 -> enter
- 删除 -> delete(捕获删除和退格键)
- 退出 -> esc
- 空格 -> space
- 换行 -> tab(特殊，必需搭配keydown使用)
- 上 -> up
- 下 -> down
- 左 -> left
- 右 -> right

2. Vue 中未提供别名的按键，可以使用按键原始的key值去绑定，但注意要转为kebab-case(短横线命名)
3. 系统修饰键(用法特殊):ctrl,alt,shift,win
   1. 配合keyup使用：按下修饰键的同时，按下其他键，随后释放其他键，事件才被触发
   2. 配合keydown使用：正常触发事件
4. 也可以使用keyCode去指定具体的按键（不推荐）
5. Vue.config.KeyCodes.自定义键名 = 键码，可以去定制按键别名

```html
<body>
    <div id="root">
        <input type="text" placeholder="按下回车提示输入" @keyup.enter="showInfo">
    </div>
</body>

<script type="text/javascript">
    Vue.config.productionTip = false

    // 创建Vue实例
    const vm = new Vue({
        el: '#root',
        data: {
            name: 'xx'
        },
        methods: {
            showInfo(event) {
                //if(event.keyCode!=13) return
                console.log(event.target.value)
            }
        },
    })
</script>
```

## 计算属性

---

1. 定义：要用的属性不存在，需要通过已有属性计算的来
2. 原理：底层借助`Object.defineproperty`方法来提供`getter`和`setter`
3. get函数什么时候执行？
   1. 初次读取时会执行一次
   2. 当依赖的数据发生改变时会被再次调用
4. 优势：与methods实现相比，内部有缓存机制，效率更高，调试方便
5. 备注：
   1. 计算属性最终会出现在vm上，直接读取使用即可
   2. 如果计算属性要被修改，那么必须写set函数去响应修改，且set中要引起计算时依赖的数据发生改变

```html
<body>
    <div id="root">
        FirstName<input type="text" v-model="firstName"><br/>
        LastName<input type="text" v-model="lastName"><br/>
        FullName<span>{{fullName}}</span>
    </div>
</body>
<script type="text/javascript">
    Vue.config.productionTip = false

    // 创建Vue实例
    const vm = new Vue({
        el: '#root',
        data: {
            firstName: '张',
            lastName:'三'
        },
        computed:{
            fullName:{
                get(){
                    return this.firstName + '-' + this.lastName
                },
                set(value){
                    const arr = value.split('-')
                    this.firstName = arr[0]
                    this.lastName = arr[1]
                }
            }
        }
    })
</script>
```

## 监视属性

---

1. 当被监视的属性变化时，回调函数自动调用，进行相关操作
2. 监视的属性必须存在，才能进行监视
3. 监视的两种写法：
   1. `new Vue` 时传入watch配置
   2. 通过`vm.$watch`监视

```html
<body>
    <div id="root">
        <h3>天气很{{weather}}</h3>
        <button @click="change">点击切换</button>
    </div>
</body>
<script type="text/javascript">
    Vue.config.productionTip = false

    // 创建Vue实例
    const vm = new Vue({
        el: '#root',
        data: {
            name: 'xx',
            isHot: true
        },
        computed: {
            weather() {
                return this.isHot ? '热' : '冷'
            }
        },
        methods: {
            change() {
                this.isHot = !this.isHot
            }
        },
        // watch: {
        //     immediate:true, //初始化让handler调用
        //     isHot: {
        //         handler(newValue, oldValue) {
        //             console.log(newValue, oldValue)
        //         }
        //     }
        // }
    })

    vm.$watch('isHot',{
        immediate: true, //初始化让handler调用
        handler(newValue, oldValue) {
            console.log(newValue, oldValue)
        }
    })
</script>
```

深度监视：
1. Vue中的watch默认不监视对象内部值的改变（一层）
2. 配置`deep:true`可以监测对象内部值改变（多层）
**Tips:**
1. Vue自身可以监测对象内部值的改变，但Vue提供的watch默认不可以
2. 使用watch时根据数据的具体结构，决定是否采用深度监视

```html
<body>
    <div id="root">
        <h4>{{numbers.a}}</h4>
        <button @click="numbers.a++">a++</button>
    </div>
</body>
<script type="text/javascript">
    Vue.config.productionTip = false

    // 创建Vue实例
    const vm = new Vue({
        el: '#root',
        data: {
            numbers: {
                a: 1,
                b: 1
            }
        },
        watch: {
            // 监视多级结构中某个属性值的变化
            'numbers.a': {
                handler() {
                    console.log('a变了')
                }
            },
            //监视多级结构中所有属性值的变化
            numbers:{
                deep:true,
                handler(){
                    console.log('numbers变了')
                }
            }
        }
    })
```

`computed`和`watch`的区别
1. `computed`能完成的`watch`都能完成
2. `watch`能完成的`computed`不一定能完成，如`watch`可以进行异步操作

**Tips:**
1. 被Vue所管理的函数，最好写成普通函数，这样this指向的才是vm或组件实例对象
2. 所有不被vue所管理的函数（定时器的回调函数，ajax的回调函数等），最好写成箭头函数，这样this等指向才是vm或组件实例对象

## 绑定样式

---

1. class 样式
   写法：`class='xxx'` xxx可以是字符串、对象、数组
   - 字符串适用于：样式多类名不确定，需要动态指定
   - 对象写法，适用于：要绑定的样式个数确定、名字也确定，但要动态决定是否使用
   - 数组写法，适用于：要绑定的样式个数不确定、名字也不确定
2. style样式
   - `:style="{fontSize:xxx}"`其中xxx是动态值
   - `:style="[a,b]"`其中a、b是样式对象

```html
<body>
    <div id="root">
        <!-- 绑定class样式——字符串，适用于：样式多类名不确定，需要动态指定 -->
        <div class="basic" :class="type" @click="changeType">{{name}}</div>
        <!-- 绑定class样式——数组写法，适用于：要绑定的样式个数不确定、名字也不确定 -->
        <div class="basic" :class="typeArr">{{name}}</div>
        <!-- 绑定class样式——对象写法，适用于：要绑定的样式个数确定、名字也确定，但要动态决定是否使用 -->
        <div class="basic" :class="typeObj">{{name}}</div>

        <!-- 绑定style样式——对象写法 -->
        <div class="basic" :style="styleObj">{{name}}</div>
        <!-- 绑定style样式——数组写法 -->
        <div class="basic" :style="styleArr">{{name}}</div>
    </div>
</body>
<script type="text/javascript">
    Vue.config.productionTip = false

    // 创建Vue实例
    const vm = new Vue({
        el: '#root',
        data: {
            name: 'xx',
            type: 'normal',
            typeArr: ['type1', 'type2', 'type3'],
            typeObj: {
                type1: false,
                type2: false,
                type3: true
            },
            styleObj: {
                fontSize: '40px',
                color: 'red'
            },
            styleObj2: {
                backgroudColor: 'orange'
            },
            styleArr: [
                {
                    fontSize: '40px',
                    color: 'red'
                },
                {
                    backgroudColor: 'orange'
                },
            ]
        },
        methods: {
            changeType() {
                const arr = ['normal', 'type1', 'type2']
                this.type = arr[Math.floor(Math.random() * 3)]
            }
        },
    })
</script>
```

## 条件渲染

---

1. `v-if`  
   写法：
   1. `v-if = "表达式"`
   2. `v-else-if = "表达式"`
   3. `v-else = "表达式"`
   适用于：切换频率较低的场景  
   特点：不展示的DOM元素直接被移除  
   注意：`v-if`可以和`v-else-if`,`v-else`一起使用，但要求结构不能被打断
2. `v-show`
   写法：`v-show = "表达式"`  
   适用于：切换频率较高的场景  
   特点：不展示的DOM元素未被移除，仅仅是使用样式隐藏掉
3. 备注：使用`v-if`时，元素可能无法获取到，而使用`v-show`一定能获取到

```html
<body>
    <div id="root">

        <!-- 使用v-show做条件渲染 -->
        <h2 v-show="false">{{name}}</h2>
        <h2 v-show="1===1">{{name}}</h2>

        <!-- 使用v-if做条件渲染 -->
        <h2 v-if="false">{{name}}</h2>
        <h2 v-if="1===1">{{name}}</h2>

        <!-- v-else-if,v-else -->
        <h2 v-if="n===1">1</h2>
        <h2 v-else-if="n===2">2</h2>
        <h2 v-else="n===3">3</h2>

        <!-- v-if和template搭配使用 -->
        <template v-if="true">
            <h1>1</h1>
            <h2>2</h2>
            <h3>3</h3>
        </template>

    </div>
</body>
```

## 列表渲染

---

`v-for`指令
1. 用于展示列表数据
2. 语法：`v-for="(item,index) in xxx" :key="yyy"`
3. 可遍历：数组、对象、字符串（用的少）、指定次数（用的少）

```html
<body>
    <div id="root">
        <ul>
            <li v-for="(p,index) in persons" :key="index">
                {{p.name}}-{{p.age}}
            </li>
        </ul>
    </div>
</body>
<script type="text/javascript">
    Vue.config.productionTip = false

    // 创建Vue实例
    const vm = new Vue({
        el: '#root',
        data: {
            name: 'xx',
            persons:[
                {id:'001',name:'张三',age:18},
                {id:'002',name:'李四',age:19},
                {id:'003',name:'王五',age:20},
            ]
        }
    })
</script>
```

key有什么作用？（key的内部原理）
1. 虚拟DOM中key的作用：
   key是虚拟DOM对象的标识，当状态中的数据发生变化时，Vue会根据 `新数据` 生成 `新的虚拟DOM`，随后Vue进行 `新虚拟DOM` 与 `旧虚拟DOM` 的差异比较，比较规则如下：
   1. 旧虚拟DOM中找到了与新虚拟DOM相同的Key：
      1. 若虚拟DOM中内容没变，直接使用之前的真实DOM
      2. 若虚拟DOM中内容变了，生成新的真实DOM，随后替换掉页面中之前的真实DOM
   2. 旧虚拟DOM中未找到与新虚拟DOM相同的Key：创建新的真实DOM，随后渲染到页面
2. 用index作为key可能会引发到问题：
   1. 若对数据进行逆序添加，逆序删除等破坏顺序的操作：会产生没有必要的真实DOM更新 ——> 界面效果没问题但效率低
   2. 如果结构中还包含输入类的DOM：会产生错误DOM更新 ——> 界面有问题
3. 开发中如何选择key：
   1. 最好使用每条数据的唯一标识作为key：id、手机号等唯一值
   2. 如果不存在对数据逆序添加、逆序删除等破坏顺序操作，仅用于渲染列表用于展示，使用index作为key没有问题

### 实现列表过滤

```html
<script type="text/javascript">
    Vue.config.productionTip = false

    // 创建Vue实例
    const vm = new Vue({
        el: '#root',
        data: {
            name: 'xx',
            keyword: '',
            persons: [
                { id: '001', name: '张三', age: 18 },
                { id: '002', name: '李四', age: 19 },
                { id: '003', name: '王五', age: 20 },
            ]
        },
        computed: {
            searchPerson() {
                return this.persons.filters((p) => {
                    return p.name.indexOf(this.keyword) !== -1
                })
            }
        }
    })
</script>
```

### 实现列表过滤并排序

```html
<script type="text/javascript">
    Vue.config.productionTip = false

    // 创建Vue实例
    const vm = new Vue({
        el: '#root',
        data: {
            name: 'xx',
            keyword: '',
            sortType: 0, // 0:原顺序，1:降序，2:升序
            persons: [
                { id: '001', name: '张三', age: 18 },
                { id: '002', name: '李四', age: 19 },
                { id: '003', name: '王五', age: 20 },
            ]
        },
        computed: {
            searchPerson() {
                const arr = this.persons.filters((p) => {
                    return p.name.indexOf(this.keyword) !== -1
                })

                // 判断是否需要排序
                if (this.sortType) {
                    arr.sort((a, b) => {
                        return this.sortType === 1 ? b.age - a.age : a.age - b.age
                    })
                }
                return arr
            }
        }
    })
</script>
```

## 数据监视

---

1. vue会监视data中所有层次的数据
2. 如何监测对象中的数据？
   通过setter实现监视，Vue默认不做响应式处理
   1. 对象中后追加的属性，Vue默认不做响应式处理
   2. 如需给后添加的属性做响应式，请使用如下API：
      - `Vue.set(target,propertyName/index,value)`
      - `vm.$set(target,propertyName/index,value)`
3. 如何监测数组中的数据？
   通过包裹数组更新元素的方法实现，本质就是做了两件事
   1. 调用原生对应的方法对数组进行更新
   2. 重新解析模版，进而更新页面
4. 在Vue修改数组中的某个元素一定要使用如下方法：
   1. 使用这些API：`push()`,`pop()`,`shift()`,``unshift()`,`splice()`,`sort()`,`reverse()`
   2. `Vue.set()`或`vm.$set()`

**Tips:** `Vue.set()`和`vm.$set()` 不能给vm或vm的根数据对象添加属性

## 收集表单数据

---

- 若`<input type="text">`，则`v-model`收集的是value值，用户输入的就是value值
- 若`<input type="radio">`，则`v-model`收集的是value值，且要给标签配置value值
- 若`<input type="checkbox">`
  1. 没有配置input的value属性，那么收集的就是checked（勾选或未勾选，布尔值）
  2. 配置input的value属性
     1. `v-model`的初始值是非数组，那么收集的就是checked（勾选或未勾选，布尔值）
     2. `v-model`的初始值是数组,那么收集的就是value组成的数组
   
**Tips:** 
`v-model`的三个修饰符  
1. `lazy` 失去焦点再收集数据
2. `number` 输入字符串转为有效数字
3. `trim` 输入首位空格过滤

```html
<body>
    <div id="root">
        <form @submit.prevent="demo">
            <label for="account">帐号：</label>
            <input type="text" id="account" v-model.trim="userInfo.account">
            <br>
            <label for="password">密码：</label>
            <input type="password" id="password" v-model="userInfo.password">
            <br>
            年龄：
            <input type="number" v-model.number="userInfo.age">
            <br>
            性别：
            男<input type="radio" name="sex" v-model="userInfo.sex" value="male">
            女<input type="radio" name="sex" v-model="userInfo.sex" value="female">
            <br>
            爱好：
            吃饭<input type="checkbox" v-model="userInfo.hobby" value="eat">
            睡觉<input type="checkbox" v-model="userInfo.hobby" value="sleep">
            打豆豆<input type="checkbox" v-model="userInfo.hobby" value="hit">
            <br>
            所属地区<br>
            <select v-model="userInfo.city">
                <option value="">请选择地区</option>
                <option value="beijing">北京</option>
                <option value="shanghai">上海</option>
                <option value="guangzhou">广州</option>
                <option value="shenzhen">深圳</option>
            </select>
            <br>
            其他信息：<br>
            <textarea v-model.lazy="userInfo.other"></textarea><br>
            <input type="checkbox" v-model="userInfo.agree">阅读并接受<a href="http://xxxerxes.github.io">用户协议</a>
            <br>
            <button>提交</button>
        </form>
    </div>
</body>
<script type="text/javascript">
    Vue.config.productionTip = false

    // 创建Vue实例
    const vm = new Vue({
        el: '#root',
        data: {
            userInfo: {
                account: '',
                password: '',
                age:'',
                sex: '',
                hobby: [],
                city: '',
                other: '',
                agree: ''
            }
        },
        methods: {
            demo() {
                console.log(JSON.stringify(this.userInfo))
            }
        },
    })
</script>
```

## 内置指令

---

### v-html

1. 作用：向指定节点中渲染包含html结构的内容
2. 与插值语法的区别：
   1. `v-html`会替换掉节点中所有内容,`{{xxx}}`不会
   2. `v-html`可以识别html结构
3. 注意！`v-html`有安全性问题
   1. 在网站上动态渲染任意HTML是非常危险的，容易导致XSS攻击
   2. 一定要在可信的内容上使用`v-html`，永远不要在用户可提交的内容上使用

### v-cloak

1. 本质是一个特殊属性，vue实例创建完毕并接管容器后，会删除`v-cloak`属性  
2. 使用css配合`v-cloak`可以解决网速慢时页面展示出`{{xx}}`的问题

```html
<style>
    [v-cloak]{
        display:none;
    }
</style>

<h2 v-cloak>{{name}}<h2>
```

### v-once

1. `v-once` 所在节点在初次动态渲染后，就视为静态内容了
2. 以后数据的改变不会引起`v-once`所在结构的更新，可以用于优化性能

## v-pre

1. 跳过其所在节点的编译过程
2. 可利用它跳过没有使用指令语法、插值语法的节点，可以加快编译