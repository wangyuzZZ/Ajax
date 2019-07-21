# 安装

```shell
yarn add -s axios
or
npm i -S axios
```

# 引入

```javascript
//在main.js中
//导入axios
import Axios from 'axios'
//挂载到vue
Vue.prototype.$axios = Axios
```

>提示：
>
>-> 可以在plugins中全局引入 axios。
>
>-> axios 请求需挂载在 **create** 生命周期钩子上。

# 示例

## 1.GET

```javascript
this.$axios.get('url', {
  params: {
      // get 参数可放置于此字段中，key-value对形式设置
      // 一般建议这么做，便于控制
      //例
      name:'wy'
      
  }
}).then(res => {
    //请求成功返回的值
  	console.log(res)
})
.catch(error => {
    //请求失败返回的值
 	 console.log(error);
})
```

## 2.post

```javascript
this.$axios.post('url', $Qs.stringify({
    // post 参数直接在第2个参数中以key-value对形式设置
    //例
    name:'admin',
    password:'123'
}))
.then(res => {
    //请求成功返回的值
  	console.log(res)
})
.catch(error => {
    //请求失败返回的值
  console.log(error);
})
```

>注意：
>
>form-data：?name=LiHongyao&age=25
>
>x-www-form-urlencode：{name=LiHongyao, age=25}
>
>axios 接受的 **post** 请求参数格式是 form-data 格式，
>
>如何将 x-www-form-urlencode 转换为 form-data 格式呢？操作如下：
>
>stips1 -> 首先你需要引入 **qs** 库，qs库无需安装，直接引入即可：
>
>可以全局引用
>
>```javascript
>//mian.js中
>import Qs from 'qs'
>//挂载
>Vue.prototype.$Qs = Qs
>```

# 全局配置

```javascript
//在main.js中
Axios.defaults.baseURL = 'localhost://8081';

//例
this.$Axios.get('localhost://8081/goods',{
    ...
})
//可替换为
this.$Axios.get('/goods',{
    ...
})    
```

全局配置的好处在于在请求当前域名下的资源时，我们无需在加域名，只需要写资源相对路径即可。

# 拦截器

在请求或响应被then或catch 处理前拦截它们

```javascript
//在main.js中
// 添加请求拦截器
Axios.interceptors.request.use(function (config) {
  // 在发送请求之前做些什么
  console.log( config)
  return config;
}, function (error) {
  // 对请求错误做些什么
  return Promise.reject(error);
});

// 添加响应拦截器
Axios.interceptors.response.use(function (response) {
  // 对响应数据做点什么
  console.log(response)
  return response;
}, function (error) {
  // 对响应错误做点什么
  return Promise.reject(error);
});
```

>为什么需要拦截器呢？拦截器可以做什么事情？
>
>我们可以通过拦截器判断参数是否合理？请求有没有问题？请求的数据是否有问题？
>
>比如，我们之前对数据进行请求时，需要对请求参数做转换，我们可以把转换的逻辑扔在拦截器中进行处理，如下所示：
>
>```javascript
>import Qs from 'qs';
>Axios.interceptors.request.use(function (config) {
>  if(config.method == 'post') {
>    config.data = Qs.stringify(config.data);
>  }
>  return config;
>}, function (error) {
>  return Promise.reject(error);
>});
>```

# 解决跨域

1.修改./config.index.js文件：

```javascript
proxyTable: {
  '/api': {
    target: 'http://api.douban.com/v2',
    changeOrigin: true,
    pathRewrite: {
      '^/api': ''
    }
  }
}
```

2.在main.js文件中添加host

```javascript
Vue.prototype.HOST = '/api'
```

3.请求数据

```javascript
let url = this.HOST + '/movie/top250';
this.$axios.get(url, {
  params: {
    count: 10,
    start: 0
  }
})
.then(res => {
  console.log(res);
}).catch(error => {
  console.log(error);
})

//后台跨越处理只需要设置 res.setHeader("Access-Control-Allow-Origin", "*");
```

> 注意：此种跨域解决方案，只适用于测试阶段，打包的时候，不具备服务器，就不能跨域了，交给后端处理。产品上线之后，就不存在跨域问题啦

> 提示：一旦修改了配置文件，需重新执行‘npm run dev’，否则配置不生效。