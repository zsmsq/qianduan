# RESTfull Api

1. get:   /资源        /users
2. post:  /资源       /users     自动维护id
3. put：/资源/id   /users/1    {username：'atguigu'}
4. patch:  /资源/id  /users/1  {username：'atguigu'}
5. delete: /资源/id  /users/1  删除id是1 用户

# axios

> axios返回值都是promise对象
>
> 发送ajax请求的库，最流行，应用最广的前端请求库
>
> node可以用
>
> react vue 都是官方推荐的



## 函数式

- method ：请求方式
- url： 地址
- 参数：
  - get：
    - 1. query参数
         1. http://localhost:8080/server?a=1&b=2
    - 2. params参数
  - post：
    - 请求体 body
- 请求头：
  - headers

## 请求分类

1. get / delete : 没有请求体
2. post / put /patch： 有请求体、



### get 请求

```js
// http://localhost:8080/server?a=1&b=2
let res = axios({
    method:'get',
    url:'http://localhost:8080/server',
    params:{
        a:1,
        b:2
    },
    headers:{
        z:1,
        y:2
    }
})
// axios函数的返回值：首先是一个 promise对象
// 结果是一个object对象
{
	data: 响应体,
    status: 状态码
    statusText：状态字符串
    request: 请求对象 xhr
    headers
    config
}
```

### post 

```js
axios({
    method:'post',
    url:'xxxxx',
    params:{
        a:1,
        b:2
    },
    // content-type: application/x-www-form-urlencode
    //data: 'name=atguigu&age=10',
    // content-type: application/json
    data: {name='atguigu',age=10}
})
```

### put 请求

```js
async function sendPut(){
            let id = 1;
            let {data} = await axios({
                method:'put',
                // params参数，只能通过拼接字符串的方式携带
                url:'http://localhost:8080/put_test/' + id,
                // put 请求本质跟post一致，也可以发送请求体，并且接收也是 request.body
                data:{username:'yuonly',age:20}
            })
            console.log(data);
        }
```

### delete请求

```js
async function sendDELETE(){
            let id = 2
            let {data} = await axios({
                method:'delete',
                url:'http://localhost:8080/delete_test/' + id,
                // 没有请求体
                data:'name=atguigu&age=10',
                params:{
                    a:1,
                    b:2
                },
                headers:{
                    z:200
                }
            })
            console.log(data);
        }
```

![image-20220816152946112](01-axios%E7%AC%94%E8%AE%B0.assets/image-20220816152946112.png)



## 对象式的使用方式

1. get:    

   ```js
   axios.get(url[, 配置对象])
   {
       //  url?a=1&b=2
       params:{
           a:1,
           b:2
       },
       headers:{
           xxx:yyy
       }
   }
   axios.get(url,config)
   
   // delete
   axios.delete(url,config)
   ```

2. post

   ```js
   axios.post(url,请求体[, 配置对象])
   
   data:请求体
   axios.post(url,data,config)
   
   //put
   axios.put(url,data,config)
   
   //patch
   axios.patch(url,data,config)
   ```

## 常见错误

1. 服务器没有启动

![image-20220816153846826](01-axios%E7%AC%94%E8%AE%B0.assets/image-20220816153846826.png)



2. 请求路径错误 【协议 、端口号、域名、路径】

   1. 前端错误： axios ---> url  跟后端不匹配，你写错了，单词拼错了

   ```js
   axios.get('http://localhost:8080/serve')
   app.get('/server',(request,response))
   ```

   2. 后端错误：压根没有定义 /server接口

![image-20220816154021427](01-axios%E7%AC%94%E8%AE%B0.assets/image-20220816154021427.png)



### 小结

- Restfull Api

  - 将数据看成资源
  - 通过请求方式来区分，是什么操作

- postman： 是一个http客户端，可以发送http请求。我们一般用它来测试后端提供的接口

- json-server:  块速搭建服务【restFull api】

  - 1. 全局安装： npm i json-server -g  ===> 会获得一个 json-server的命令
    2. 创建数据： xxx.json

    ```js
    {
        "computers":[
            {
                id:自动维护,
                自定义的字段: 值
            }
        ]
    }
    ```

    3. 启动json-server 服务

    ```shell
    # 在 xxx.json文件目录中打开 终端
    json-sever --watch xxx.json
    ```

- axios

  - 函数式

    - config：配置对象

      ```js
      {
          method:请求方式,
          url：请求地址,
          params：query参数,
          data: 请求体数据 ,
          headers:请求头
      }
      
      axios({
          method:'get',
          url:'xxxx',
          ..
          ..
          ..
      })
      ```

      

  - 对象式

    - 1. get： axios.get(url [, config])
      2. delete: axios.delete(url [,config])
      3. post: axios.post(url, 请求体[,config])
      4. put:axios.post(url, 请求体[,config])
      5. patch:axios.post(url, 请求体[,config])

- axios 返回值：

  - promise对象：
  - 请求回来：成功的结果值也是一个对象

  ```js
  {
      status:
      statusText
      config
      headers
      request
      
      data:服务器端响应回来的结果
  }
  ```

  

​		

## 拦截器

1. 请求拦截器【先添加的后执行 unshift】
2. 响应拦截器【按顺序执行 push】

![image-20220817101354475](01-axios%E7%AC%94%E8%AE%B0.assets/image-20220817101354475.png)

3. 执行的顺序：

   如果是成功的： 请求拦截器成功2 -》请求拦截器成功1 -》 dispatchRequest -> 响应拦截器成功1 ---> 响应拦截器成功2 ---> axios.then(成功的回调)

   如果是失败的：请求拦截器成功2 后面如何执行，完全取决于 请求拦截器成功2的 promise状态

### 拦截器的应用

1. 请求拦截

   1. 携带公共的头部信息
   2. 可以配置baseURL
   3. 根据业务需求，配置信息 》。。。。

   ```js
   axios.interceptors.request.use(config => {
               // 可以配置公共的头部
               config.headers.token = 'xxxxxxyyyyzzz'
               // 配置公共的url
               config.baseURL = 'http://localhost:8080'
               return config;
           }, error => {
               // Do something with request error
               return Promise.reject(error);
           });
   ```

   

2. 响应拦截器：处理响应回来的数据。

   1. return response.data

```js
// 响应拦截器. 可以处理响应数据
        axios.interceptors.response.use(response=>{

            return response.data
        },error=>{
            return Promise.reject(error)
        })


// 直接通过response获取响应体
        axios.get('/server').then(response=>{
            console.log(response)
        })
```



## 取消请求

> 通过一个配置项 cancelToken

```js
let cancel = null;
// 对象式的
axios.get('/server',{
    cancelToken:new axios.CancelToken(c=>{
        cancel = c
    })
})
// 函数式
axios({
    cancelToken:new axios.CancelToken(c=>{
        cancel = c
    })
})

{
    method
    url
    params
    data
    headers
    cacelToken
    timeout
}
```



1. 1. 

# 附录

## 用户代码片段配置

1. 文件  ---> 首选项  ---> 配置用户代码片段
2. 点击 ---> 新代码片段 --> 输入代码片段名--> 回车
3. 配置

```js
"axios_test": {
    // scope：代码片段生效的文件类型
    "scope": "javascript,typescript",
    // prefix：代码片段唤醒词
    "prefix": "log",
    // body：通过唤醒词快速创建的代码片段。$1 光标第一次停留的位置 $2 按tab件第二次停留的位置
    "body": [
        "console.log('$1');",
        "$2"
     ],
     "description": "Log output to console"
}
```

### 如何找到用户代码片段位置？

1. 查看 ---> 显示导航痕迹
2. 按照导航痕迹路径，找到对应文件进行修改或删除

![image-20220816113258585](01-axios%E7%AC%94%E8%AE%B0.assets/image-20220816113258585.png)



## 关于复读

> 目标：入行称为程序员，找一个高薪的工作

1. vue阶段在重学

   因为找工作全靠vue

   因为你工作用什么技术栈，不是你决定的。

   公司招人的时候，按 vue招的很多，不代表上班之后用的就是vue

   node  express  react

2. 不想重学

   1. 完全听不懂： 立刻马上，right now 重学。基础太差了。

   2. 不懂的越来越多

3. 能听懂，但是自己写不出来的。

   1. 多练

4. 编程不是老师教会的。80% ，都是靠自己练。

   





















