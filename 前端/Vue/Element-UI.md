# Element UI

1. npm安装

    npm i element-ui
    npm install babel-plugin-component -D

2. 若引入所有样式，在`main.js`中写入以下内容
   ```javascript
    // 引入ElementUI组件库
    import ElementUI from 'element-ui';
    // 引入ElementUI全部样式
    import 'element-ui/lib/theme-chalk/index.css';

    Vue.use(ElementUI);
   ```
3. 若引入部分样式，在`babel.config.js`中修改为如下
   ```javascript
    module.exports = {
        presets: [
            '@vue/cli-plugin-babel/preset',
            ["@babel/preset-env", { "modules": false }],
        ],
        plugins: [
            [
            "component",
            {
                "libraryName": "element-ui",
                "styleLibraryName": "theme-chalk"
            }
            ]
        ]
    }
   ```
4. 将`main.js`改为以下内容
   ```javascript
    import {Button,Select} from 'element-ui';

    Vue.component(Button.name,Button);
    Vue.component(Select.name,Select);
    //或写为
    Vue.use(Button);
    Vue.use(Select);
   ```