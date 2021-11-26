---
title: axios 学习笔记
date: 2021-05-19 20:40:53
tags:
- 前端
- axios
- JavaScript
categories:
- 前端
- 前端工具
---

## 1.概念

- Axios 简单的理解就是 ajax 的封装。
- Axios 是一个基于 promise 的 HTTP 库。
- 支持 node 端和浏览器端。
- 使用 Promise 管理异步，告别传统 callback 方式。
- 丰富的配置项，支持拦截器等高级配置。
- 转换请求数据和响应数据。

<!-- more -->

## 2.axios 入门应用

发送 GET 请求

```js
axios({
    url: 'http://localhost/axios/api.php?a=1111&b=2222',
    method: 'get',
    params: {
        name: 'username',
    }
}).then(res => {
    console.log(res);
});
```

发送 POST 请求

```js
axios({
    method: 'post',
    url: 'http://localhost/axios/api.php',
    headers: {
        'content-type': 'application/x-www-form-urlencoded'
    },
    data: {
        name: 'username',
        age: '30',
        sex: 'aaa'
    }
}).then(res => {
    console.log(res);
});
```

## 3.axios 的 post 和 get 请求方式

```js
import axios from 'axios';

axios.request(config)
axios.get(url[, config])
axios.delete(url[, config])
axios.head(url[, config])
axios.post(url[, data[, config]])
axios.put(url[, data[, config]])
axios.patch(url[, data[, config]])


axios.get('/user?ID=12345')

axios.get('/user', {
    params: {
        ID: 12345
    }
})

axios.post(url[, data[, config]])

axios.post('/user', {
    firstName: 'Fred',
    lastName: 'Flintstone'
})
```

## 4.axios 的全局配置方案

```js
axios.default.baseURL = "http://127.0.0.1";
axios.default.timeout = 5000;
axios.default.headers.post['content-type'] = 'application/x-www-form-urlencoded';

// 整理数据
axios.defaults.transformRequest = function (data) {
    data = JSON.stringify(data);
    return data;
};
```

## 5.axios 的实例封装

有时候后台接口地址有多个并且超时时长不一样，我们不可能在 axios 中把每个后台请求的域名地址都拼接在 URl 中，并且在 axios 中的 config 写不同的超时时长，很繁琐，这个时候可以用到 axios 实例，在实例中可以配置这两种参数。

假如新建了一个 axios 实例但是没有参数，取得就是全局的配置值，实例中如果有则优先取实例中的。

axios 实例的相关配置（config 参数）

```
baseURL：请求的域名基本地址（http://localhost:8080）
timeout：后端定义的超时时长（默认是 1000ms）
url：请求的路径（如：/data.json）
method：请求方法（get、post.....）
headers：设置请求头
params：请求的参数拼接在 url 中
data：请求的参数放在 request body 中
```

```js
//create创建一个新的实例对象
var instance = axios.create({
    url: 'http://jsonplaceholder.typicode.com/users',
    imeout: 3000,
    method: 'post'
});

//即可调用方法，和axios实例同
instance.get('http://jsonplaceholder.typicode.com/users').then(Response => {
    console.log(Response);
});
```

## 6.axios 的拦截器应用

- 为每个请求都带上的参数，比如 token，时间戳等。

- 对返回的状态进行判断，比如 token 是否过期。

```js
// 请求拦截器
axios.interceptors.request.use(
    config => {
        // 每次发送请求之前判断是否存在 token
        // 如果存在，则统一在 http 请求的 header 都加上 token，这样后台根据 token 判断你的登录情况
        // 即使本地存在 token，也有可能 token 是过期的，所以在响应拦截器中要对返回状态进行判断
        const token = window.localStorage.getItem("token");
        token && (config.headers.Authorization = token);
        return config;
    },
    error => {
        return Promise.error(error);
    }
);
```

```js
axios.interceptors.response.use(
    response => {
        // 如果返回的状态码为200，说明接口请求成功，可以正常拿到数据
        // 否则的话抛出错误
        if (response.status === 200) {
            return Promise.resolve(response);
        } else {
            return Promise.reject(response);
        }
    },
    // 服务器状态码不是2开头的的情况
    // 这里可以跟你们的后台开发人员协商好统一的错误状态码
    // 然后根据返回的状态码进行一些操作，例如登录过期提示，错误提示等等
    // 下面列举几个常见的操作，其他需求可自行扩展
    error => {
        if (error.response.status) {
            return Promise.reject(error.response);
        }
    }
);
```

