---
title: Xposed+HTTP实现加解密调用
published: 2024-05-03
description: ''
image: ''
tags: ['爬虫','逆向','Android','Xposed']
category: ''
draft: false 
---




# Intro

最近在逆向某个app，抓包抓到的是加密的数据。

![image-20240503205050158](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//img/image-20240503205050158.png)

定位到解密函数，是在native层![image-20240503205130335](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//img/image-20240503205130335.png)

直接逆向so有难度![image-20240503205207767](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//img/image-20240503205207767.png)

所以打算使用巧妙的方法，主动调用解密函数。

# 实现

首先引入nanohttpd服务器依赖

```kotlin
implementation 'org.nanohttpd:nanohttpd:2.3.1'
```

```java
public class HttpServer extends NanoHTTPD{
    Gson gson = new Gson();

    public HttpServer(int port) {
        super(port);
        try{
            Class<?> clazz = Main.classLoader.loadClass("com.xxx.security.SecurityCipher");
            Constructor<?> constructor = clazz.getConstructor(VivoHook.classLoader.loadClass("android.content.Context"));
            constructor.setAccessible(true);
            Main.instance = constructor.newInstance(VivoHook.context);
            Main.aesDecryptBinary = clazz.getDeclaredMethod("decodeBinary",byte[].class);
            Main.encodeUrlParams = clazz.getDeclaredMethod("encodeUrlParams", Map.class);

        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    @SneakyThrows
    @Override
    public Response serve(IHTTPSession session) {
        Map<String, String> params = new HashMap<>();
        session.parseBody(params);
        switch (session.getUri()) {
            case "/Main/decrypt" : {
                String data = params.get("postData");
                XposedBridge.log("data: " + data);
                byte[] ret = (byte[]) Main.aesDecryptBinary.invoke(Main.instance, data.getBytes());
                return newFixedLengthResponse(new String(ret));
            }
            case "/Main/encrypt" : {
                String data = params.get("postData");
                Type type = new TypeToken<Map<String, String>>(){}.getType();
                Map<String, String> map = gson.fromJson(data, type);
                Map<String,String> invoke = (Map<String, String>) Main.encodeUrlParams.invoke(Main.instance, map);
                return newFixedLengthResponse(gson.toJson(invoke));
            }
            case "/" : {
                return newFixedLengthResponse("It's works");
            }
        }
        return super.serve(session);
    }
}
```

```java
new HttpServer(8888).start();
```

然后调用start方法启动服务器。

这里要注意有些app是多进程的，handleLoadPackage方法会调用多次，导致start会调用多次，

所以在一开始要判断进程名

```java
        if (!param.processName.equals("com.xxx")) {
            return;
        }
```

服务器开起来之后，adb连接电脑，然后进行端口转发

```cmd
adb forward tcp:8888 tcp:8888
```

然后浏览器访问web服务器，应该能看到提示

![image-20240503210251176](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//img/image-20240503210251176.png)

# 调用

![image-20240503210416089](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//img/image-20240503210416089.png)

![image-20240503210642587](https://imgbed-1254007525.cos.ap-nanjing.myqcloud.com//img/image-20240503210642587.png)

成功调用。