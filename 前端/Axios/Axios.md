# Axios

基本使用
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <link href="https://cdn.bootcdn.net/ajax/libs/twitter-bootstrap/3.3.7/css/bootstrap.min.css" rel="stylesheet">
    <script crossorigin="anonymous" src="https://cdn.bootcdn.net/ajax/libs/axios/0.27.2/axios.min.js"></script>
</head>

<body>
    <div class="container">
        <h2 class="page-header">基本使用</h2>
        <button class="btn btn-primary">GET</button>
    </div>
    <script>
        const btns = document.getElementsByTagName('button')

        // 默认配置
        axios.defaults.method = 'GET';
        axios.defaults.baseURL = 'http://localhost:5038';
        axios.defaults.params = { id: 100 };
        axios.defaults.timeout = 3000;

        btns[0].onclick = function () {
            axios({
                url: '/weatherforecast'
            }).then(response => {
                console.log('response')
            })
        }
    </script>
</body>

</html>
```

## 创建实例对象发送请求

```html
<script>
    const demo = axios.create({
        baseURL: 'http://localhost:5038',
        timeout: 2000
    });

    // 方法一
    demo({
        url: '/weatherforecast'
    }).then(response => {
        console.log(response.data)
    })

    // 方法二
    demo.get('/weatherforecast').then(response=>{
        console.log(response.data)
    })
</script>
```