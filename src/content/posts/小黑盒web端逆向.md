---
title: 小黑盒web端逆向
published: 2024-05-14
description: ''
image: ''
tags: []
category: ''
draft: false 
---
# 小黑盒web端逆向

##  Intro

最近想写个小黑盒爬虫，研究了下，![image-20240514192201251](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//img/image-20240514192201251.png)

是有加密的，发送请求必须要携带hkey和nonce这两个参数。
定位到生成这两个参数的代码，![image-20240514192742151](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//img/image-20240514192742151.png)

是在native层，在接下去逆向有些难度。

然后又发现小黑盒的部分模块是有web端的。比如，在一个游戏详情界面，点分享，然后得到的链接，就是小黑盒的网页版。

![image-20240514193509586](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//img/image-20240514193509586.png)

网页端逆向起来就方便多了。

##  算法分析

f12开发者工具随便点开一个请求，然后查看请求堆栈，![image-20240514193900131](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//img/image-20240514193900131.png)

虽然经过了混淆，但是大致能猜出来这是axios，并且参数里面没有我们要找的nonce和hkey，尝试了直接在js文件里搜nonce和hkey，都没搜到。

最后通过axios的拦截器，找到了生成这两个参数的位置。

![image-20240514194213155](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//img/image-20240514194213155.png)

原来是用了字符拼接的方法，怪不得搜不到。![image-20240514194307806](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//img/image-20240514194307806.png)

通过对代码的简单整理

```js
a = ~~(new Date().getTime/1000)) //获取秒级时间戳
o = md5(a + Math.random((new Date).getTime()).toString()) //a的值加上一个随机数，然后进行md5运算
n['nonce'] = o
n['hkey'] = v(path,a,0); //path为请求路径
```

只要解决了v函数，就算是破解完了

```javascript
        function v(t, e, n) {
            t = "/".concat(t.split("/").filter((function(t) {
                return t
            }
            )).join("/"), "/");
            for (var a = "", i = "JKMNPQRTX1234OABCDFG56789H", r = g()((n + i).split("").filter((function(t) {
                return /[0-9]/.test(t)
            }
            )).join("")), o = g()(e + t + r).split("").filter((function(t) {
                return /[0-9]/.test(t)
            }
            )).slice(0, 9).join(""), c = 0; c < 9 - o.length; c++)
                o += "0";
            for (var s = Number(o), l = 0; l < 5; l++) {
                var u = s % 26;
                s = ~~(s / 26),
                a += i[u]
            }
            var d = "".concat(function(t) {
                return t.reduce((function(t, e, n) {
                    return t + e
                }
                ))
            }(p(a.slice(-4).split("").map((function(t) {
                return t.charCodeAt()
            }
            )))) % 100);
            return d.length < 2 && (d = 0 + d),
            d = a + d
        }
```

手动简化后如下

```javascript
function hkey(path, time, n) {
    // t = /game/wx_share/info/v2/
    var i = 'JKMNPQRTX1234OABCDFG56789H'
    var r = md5(trimNumber(n + i).join(''))
    var o = trimNumber(md5(time + path + r)).slice(0, 9).join('')
    var s = Number(o)
    var a = ''
    for (var l = 0; l < 5; l++) {
        var u = s % 26;
        s = ~~(s / 26), //取整数
            a = a.concat(i[u])
    }
    var arr = p(a.slice(-4).split("").map((function (t) {
        return t.charCodeAt()
    }
    ))) //取a的后四位的Unicode值作为数组的四个元素传给p函数，把经过处理的数组求和再对100求余 即为d的值
    var d = (arr[0] + arr[1] + arr[2] + arr[3]) % 100
    return a + d
}
function trimNumber(n) {
    return n.split("").filter(t => {
        return /[0-9]/.test(t) //提取数字
    })
}
```

## 完整代码

```javascript

function u(t) {
    return 128 & t ? 255 & (t << 1 ^ 27) : t << 1
}
function d(t) {
    return u(t) ^ t
}
function h(t) {
    return d(u(t))
}
function f(t) {
    return h(d(u(t)))
}
function m(t) {
    return f(t) ^ h(t) ^ d(t)
}
function p(t) {
    var e = [0, 0, 0, 0];
    return e[0] = m(t[0]) ^ f(t[1]) ^ h(t[2]) ^ d(t[3]),
        e[1] = d(t[0]) ^ m(t[1]) ^ f(t[2]) ^ h(t[3]),
        e[2] = h(t[0]) ^ d(t[1]) ^ m(t[2]) ^ f(t[3]),
        e[3] = f(t[0]) ^ h(t[1]) ^ d(t[2]) ^ m(t[3]),
        t[0] = e[0],
        t[1] = e[1],
        t[2] = e[2],
        t[3] = e[3],
        t
}
import { createHash } from 'node:crypto'
function md5(content) {
    return createHash('md5').update(content).digest('hex')
}
function hkey(path, time, n) {
    // t = /game/wx_share/info/v2/
    var i = 'JKMNPQRTX1234OABCDFG56789H'
    var r = md5(trimNumber(n + i).join(''))
    var o = trimNumber(md5(time + path + r)).slice(0, 9).join('')
    var s = Number(o)
    var a = ''
    for (var l = 0; l < 5; l++) {
        var u = s % 26;
        s = ~~(s / 26), //取整数
            a = a.concat(i[u])
    }
    var arr = p(a.slice(-4).split("").map((function (t) {
        return t.charCodeAt()
    }
    ))) //取a的后四位的Unicode值作为数组的四个元素传给p函数，把经过处理的数组求和再对100求余 即为d的值
    var d = (arr[0] + arr[1] + arr[2] + arr[3]) % 100
    return a + d
}
function trimNumber(n) {
    return n.split("").filter(t => {
        return /[0-9]/.test(t) //提取数字
    })
}



var time = ~~(new Date().getTime() / 1000)
var nonce = md5(time + Math.random((new Date).getTime()).toString()).toLocaleUpperCase()
console.log(time)
console.log(hkey('/game/get_game_detail/', time+1, nonce))
console.log(nonce)
```

nonce的计算 是当前时间加上一个随机浮点数做md5运算，其实这个nonce可以是任意md5值，hkey的计算用到了nonce，只要nonce和hkey的nonce参数对的上就可以
