# vue-router

`vue-router`是vue的一个插件库，专门用来实现`SPA`应用  
`SPA`应用：
1. 单页Web应用
2. 整个应用只有一个完整的页面
3. 点击页面的导航链接不会刷新页面，只会做页面的局部更新
4. 数据需要通过ajax请求获取

## 创建一个路由器
1. 在src路径下新建一个`router`文件夹，并在文件夹内新建`index.js`文件
2. 在`index.js`中添加如下配置
   ```javascript
    // 该文件用于创建整个应用的路由器
    import VueRouter from 'vue-router'
    // 引入组件
    import Home from '../components/Home'
    import About from '../components/About'

    // 创建并暴露一个路由器
    export default new VueRouter({
        routes:[
            {
                path:'/about',
                component:About
            },
            {
                path:'/home',
                component:Home
            }
        ]
    })
   ```
3. 在`main.js`中引入路由器
   ```javascript
    import VueRouter from 'vue-router'
    import router from './router'

    Vue.use(VueRouter)

    new Vue({
    render: h => h(App),
    router
    }).$mount('#app')
   ```
4. 实现切换(active-class可配置高亮样式)
   ```html
    <router-link active-class="active" to="/home">Home</router-link>
   ```
5. 指定展示位置
   ```html
   <router-view></router-view>
   ```

**Tips:**
1. 路由组件通常存放在`pages`文件夹，一般组件通常存放在`components`文件夹
2. 通过切换，隐藏了的路由组件，默认是被销毁的，需要的时候再重新挂载
3. 每个组件都有自己的`$route`属性，里面存储着自己的路由信息
4. 整个应用只有一个`router`可以通过组件的`$router`属性获取

## 路由嵌套

---

```javascript
routes:[
    {
        path:'/home',
        component:Home,
        children:[
            {
                path:'message',
                component:Message,
            }
        ]
    }
]
```

跳转（写完整路径）

    <router-link to="/home/message">msg</router-link>

## 路由的query参数

---

传递参数：
```html
<!-- 跳转并携带query参数，to的字符串写法 -->
<route-link :to="`/home/message/detail?id=${id}&$title={title}`">跳转</route-link>
<!-- 跳转并携带query参数，to的对象写法 -->
<route-link
    :to="{
        path:'/home/message/detail',
        query:{
            id:001,
            title:title
        }
    }">跳转</route-link>
```

接收参数:

    $route.query.id  
    $route.query.title

### 命名路由

用于简化路由的跳转
1. 给路由命名
```javascript
routes:[
    {
        path:'/home',
        component:Home,
        children:[
            {
                name:'hello', //给路由命名
                path:'message',
                component:Message,
            }
        ]
    }
]
```
2. 简化跳转
   ```html
    <!-- 简化前，需要写完整路径 -->
    <route-link :to="/home/message">跳转</route-link>
    <!-- 简化后，直接通过名字跳转 -->
    <route-link :to="{name:'hello'}">跳转</route-link>
    <!-- 简化配合传递参数 -->
    <route-link
    :to="{
        name:'hello',
        query:{
            id:001,
            title:title
        }
    }">跳转</route-link>
    ```

## 路由的params参数

---

1. 配置路由，声明接收的`params`参数
    ```javascript
    routes:[
        {
            path:'/home',
            component:Home,
            children:[
                {
                    name:'hello', //给路由命名
                    path:'message/:id/:name', // 使用占位符声明表示接收params参数
                    component:Message,
                }
            ]
        }
    ]
    ```
2. 传递参数
```html
<!-- 跳转并携带参数，字符串写法 -->
<route-link :to="/home/message/001/xx">跳转</route-link>
<!-- 跳转并携带参数，对象写法 -->
<route-link :to="{
    name:'hello',
    params:{
        id:001,
        name:xx
    }
}">跳转</route-link>
```
**Tips:** 路由携带params参数时，若使用to的对象写法，则不能使用path配置项，必须使用name配置

3. 接收参数

    $route.params.id  
    $route.params.name

## 路由的props配置

---

让路由组件更方便的接收到参数

```javascript
{
    name:'hello',
    path:'detail/:id',
    component:Detail,

    // props值为对象，该对象中所有的key-value的组合最终都会通过props传给Detail组件
    props:{a:100}

    // props值为布尔值，布尔值为true，把路由收到的所有params参数通过props传给Detail组件
    props:true

    // props值为函数，该函数返回的对象中每一组key-value都会通过props传给Detail组件
    props(route){
        return {
            id:route.query.id,
            title:route.query.title
        }
    }
}
```

## `<router-link>`的replace属性

---

1. 作用：控制路由跳转时操作浏览器历史记录的模式
2. 浏览器的历史记录有两种写入方式，分别为`push`（追加历史记录）和`replace`（替换当前记录），路由跳转时默认为`push`
3. 开启`replace`模式：`<router-link replace ......></router-link>`

## 缓存路由组件

---

让不展示的路由组件保持挂载，不被销毁
```html
<keep-alive include="News">
    <router-view></router-view>
</keep-alive>
```

## `activated`和`deactivated`

---

路由组件所独有的两个钩子，用于捕获路由组件的激活状态

`activated`: 路由组件被激活时触发
`deactivated`: 路由组件失活时触发

## 路由守卫

---

用于对路由进行权限控制

分类：全局守卫、独享守卫、组件内守卫

### 全局守卫
```javascript
// 全局前置守卫：初始化时执行，每次路由切换前执行
router.beforeEach((to,from,next)=>{
    if(to.meta.isAuth){ // 判断当前路由是否需要进行权限控制
        if(localStorage.getItem('name')==='xx'){// 权限控制的具体规则
            next() // 放行
        }
        else{
            alert('无权查看')
        }
    }
    else{
        next() // 放行
    }
})

// 全局后置守卫：初始化时执行，每次路由切换后执行
router.afterEach((to,from)=>{
    if(to.meta.title){
        document.title = to.meta.title // 修改网页title
    }
    else{
        document.title = 'demo'
    }
})
```

### 独享守卫
```javascript
// 放在routes里边
router.beforeEnter((to,from,next)=>{
    if(to.meta.isAuth){ // 判断当前路由是否需要进行权限控制
        if(localStorage.getItem('name')==='xx'){// 权限控制的具体规则
            next() // 放行
        }
        else{
            alert('无权查看')
        }
    }
    else{
        next() // 放行
    }
})
```

### 组件内守卫
```javascript
// 进入守卫，通过路由规则，进入该组件时被调用
beforeRouteEnter(to,from,next) {

},
// 离开守卫，通过路由规则，离开该组件时被调用
beforeRouteLeave(to,from,next) {
    
}
```

## 路由器的两种工作模式

---

1. 在url中`#`及其后面的内容就是hash值
2. hash值不会包含在HTTP请求中，即：hash值不会带给服务器
3. `hash`模式
   1. 地址中带着`#`不美观
   2. 若将地址通过第三方手机app分享，当app校验严格时，地址会被标记为不合法
   3. 兼容性较好
4. `history`模式
   1. 地址干净美观
   2. 兼容性相比`hash`模式略差
   3. 应用部署上线时需要后端人员支持，解决刷新页面服务端404的问题