## 环境

---

操作系统：MacOS(Arm64)
开发工具：vscode,Visual Studio,MySQL
前端：Vue3,TypeScript,SCSS,Element Plus,Router,axios,Vuex
后端：.NET6,AutoMapper,Autofac,Sql sugar,JWT,Log4Net

**Tips:** npm可能下载速度比较慢，可以安装cnpm替代

### TypeScript

打开vue项目的路径，命令行输入`vue add typescript`  
安装时会有几个选择项，除了第一个选否，其他都为是

### sass
用于方便写样式  
`npm install sass-loader node-sass --save`，但是在arm64环境中会报错所以改为`npm install sass`

### element-plus

`npm install element-plus --save`

项目`main.js`中导入

    import ElementPlus from 'element-plus'
    const app = createApp(App)
    app.use(ElementPlus)

### 路由的配置和使用

用于URL地址和页面的适配

`npm install vue-router@next --save`

项目`main.js`中导入

    import router from './'
    const app = createApp(App)
    app.use(ElementPlus)

```javascript
// 配置路由

import { createRouter, createWebHistory } from 'vue-router'
import HomePage from "./views/HomePage.vue"
import LoveFlower from "./views/LoveFlower.vue"
import BirthdayFlower from "./views/BirthdayFlower.vue"
import FriendFlower from "./views/FriendFlower.vue"
import WeddingFlower from "./views/WeddingFlower.vue"
import FlowerDetail from "./views/FlowerDetail.vue"
import FlowerPay from "./views/FlowerPay.vue"
import Personcenter from "./views/PersonCenter.vue"

const router = createRouter({
    history: createWebHistory(),
    routes: [
        { path: '/', component: HomePage },
        { path: '/loveflower', component: LoveFlower },
        { path: '/birthdayflower', component: BirthdayFlower },
        { path: '/friendflower', component: FriendFlower },
        { path: '/weddingflower', component: WeddingFlower },
        { path: '/detail', component: FlowerDetail },
        { path: '/pay', component: FlowerPay },
        { path: '/personcenter', component: Personcenter },
    ],
})

export default router
```

## 网站模块划分

---

主页面
1. 网站首页 HomePage
2. 4个商品列表页 LoveFlower,BirthdayFlower,FriendFlower,WeddingFlower
3. 详情页 Detail
4. 支付页 Pay
5. 个人中心 PersonCenter

组件
1. 网站首位组件 HeaderCom,FooterCom
2. 登录注册组件 LoginCom,RegisterCom
3. 首页主体模块组件 HomeContent
4. 商品列表组件 FlowerList