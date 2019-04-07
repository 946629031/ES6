# Promise 对象

异步：操作之间没啥关系，可同时进行多个操作，代码更复杂<br>
同步：同时只能做一件事，代码简单



## 一、历史遗留问题
> 在讲解什么是 Promise 之前，我们先来看看为什么有会出现 Promise 这个对象。我们先来看看过去，都是怎么处理多个异步操作的。


举个例子：我们打开 taobao.com ，看到淘宝的首页，里面的所有数据都不是直接写死在前端页面上的，而是所有数据都通过异步请求 动态加载到页面上的。那么问题来了，所有的页面数据，包括各种banner图，产品分类，各个广告图的数据... 数据那么多，那么多模块，那么多部分，一个部分套着那么多数据，都是怎么通过异步请求实现动态加载的呢？


1. 我们通过逻辑思路推理，大致是通过下面代码实现异步加载的：
```js
ajax('/banners', function(){}, function(){})
// 我们假设有那么一个方法 异步请求 第一个function()是success执行，第二个function()是error执行


ajax('/banners', function(banner_data){
    ajax('/hotItems', function(hotItem_data){
        ajax('/slides', function(slide_data){
            ajax('/slides', function(slide_data){
    
            }, function(){
                alert('读取失败')
            })
        }, function(){
            alert('读取失败')
        })
    }, function(){
        alert('读取失败')
    })
}, function(){
    alert('读取失败')
})
```
上面的代码，只是异步请求了4个模块，一个套一个的异步请求，代码就如此的复杂了。
由此可见，异步请求虽然用户体验好，但是却让代码跟复杂了。




2. 我们再来看看同步请求的方式
```js
let banner_data = ajax_async('/banners');
let hotItem_data = ajax_async('/hotItems');
let slide_data = ajax_async('/slides');
let banner_data = ajax_async('/banners');
```
由此可见，同步方式虽然体验不好，容易卡页面，但是它的代码确实更加的简单





## 二、那么问题来了
> 问：有没有一种方法，既能简化代码，又能不卡页面呢？<br>
> 答：有的，这就是我们的ES6中的 Promise

## Promise写法
```js

let p = new Promise(function(resolve, reject){
    // 异步代码
    // resolve  成功了
    // reject   失败了

    $.ajax({
        url: 'arr.txt',
        dataType: 'json',
        success(arr){
            resolve(arr)
        },
        error(err){
            reject(err)
        }
    })
})

// 那么定义完Promise方法了，我们该怎么调用呢？
p.then(function(){
    alert('成功了')
}, function(){
    alert('失败了')
})
// 通过Promise.then方法调用
// 第一个function()实际上就是resolve 成功时执行，
// 第二个function()实际上就是reject 失败时执行

```

> arr.txt文件<br>
> [12,3,6,798,123,"text"]

> json.txt文件<br>
> {"a":12, "b":6, "c":"this is a text"}





## 三、那么该如何请求多个数据呢？

### 1. 请求多个数据
我们首先想到的方法是
```js
let p1 = new Promise(function(resolve, reject){
    $.ajax({
        url: 'arr.txt',
        dataType: 'json',
        success(arr){
            resolve(arr)
        },
        error(err){
            reject(err)
        }
    })
})

let p2 = new Promise(function(resolve, reject){
    $.ajax({
        url: 'json.txt',
        dataType: 'json',
        success(arr){
            resolve(arr)
        },
        error(err){
            reject(err)
        }
    })
})

p1.then(function(){
    alert('成功了')

    p2.then(function(){})  // 这样...？？？   很显然，这不是一个很好的方法
}, function(){
    alert('失败了')
})

```



## 2.其实, ajax是有返回值的，它就是 promise对象
```js
let p = $.ajax({url: 'arr.txt', dataType: 'json'})
console.log(p)  // 其实ajax的返回值就是 promise
```



<br><br>

最后，结论来了

# 2. Promise的最简写法
```js
Promise.all([
    $.ajax({url: 'arr.txt',  dataType: 'json'}),    // 要异步请求的数据
    $.ajax({url: 'json.txt', dataType: 'json'})
]).then(function(results){          // 全部请求成功时执行
    let [arr, json] = results;
    console.log(results)
    alert('成功了')
}, function(){                      // 如果至少有一个失败的话执行
    alert('失败了')
})
```

<br><br>


# Promise.race()   竞速

* Promse.race就是赛跑的意思
* 哪个结果获得的快，就返回那个结果
* 不管结果本身是成功状态还是失败状态

```js
Promise.race([
    $.ajax({url: 'http://a2.taobao.com/data/user'}),    // 负载点1
    $.ajax({url: 'http://a3.taobao.com/data/user'}),    // 负载点2
    $.ajax({url: 'http://a15.taobao.com/data/user'}),   // 负载点3
    $.ajax({url: 'http://a9.taobao.com/data/user'})     // 负载点4
])
```