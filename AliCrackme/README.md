## 阿里CTF比赛之AliCrackme逆向分析

先知：图片弱看不了，请点击我去国内的码云查看：[https://gitee.com/qqizai/CrackApp](https://gitee.com/qqizai/CrackApp)

想要了解最新动态，敬请关注[我的博客](https://blog.csdn.net/weixin_41173374)

### 本文将介绍两种方法：frida hook关键函数、so动态调试



#### 一、frida hook关键函数

首先介绍一下，我分析的思路，当接到一个任务需要逆向某APP，分析步骤如下：
- 1.首先体验一遍APP
- 2.体验之后，针对公司所给的需求/或者我们所需要的目标进行针对性的观察，再次体验一遍流程
- 3.抓包分析，提交的数据
- 4.APP是否防抓包、请求参数中是否需要特殊的加密
- 5.脱壳、反编译
- 6.针对关键词搜索代码，进而分析代码逻辑
- 7.找到加密逻辑之后，是否能够使用Python/Java/JS等语言能够实现还原的
- 8.若加密在so文件中，那么需要分析so文件里面的加密逻辑
- 9.这时候就考验真正的逆向技术了，不是开玩笑的 ----> 敬请欣赏一波搞笑视频：[一阵我放低你，就好嗨无面噶喔，唔系讲笑噶喔](https://www.bilibili.com/video/av668786650/)
- 10.使用IDA打开so文件分析
- 11.有必要的话，需要使用IDA进行动态调试
- 12.此时可能会遇到APP的反调试、so文件的反调试
- 13.这时候需要解决反调试才能进行后面的动态调试的分析

写此文章的时候，笔者认为第8步开始就是算难题，**需要给点耐心才能分析**

各位**打工人、干饭人**加油哟，未来可期！！！

##### 1.使用 jadx 反编译，发现没有壳，是可以正常反编译的，而且还没有混淆

![01](https://github.com/qqizai/CrackApp/tree/master/AliCrackme/statics/alicrackme01.jpg)
- 首先查看 AndroidManifest.xml，找到启动页加载的方法

![02](https://github.com/qqizai/CrackApp/tree/master/AliCrackme/statics/alicrackme02.jpg)
- 找到对应的源码，分析里面的逻辑
- 1首先引入眼帘的是：“验证码校验失败” 
- 2网上稍微看一下下，看到判断逻辑，若为true执行if里面的
- 3再细看，看到一个方法：securityCheck，如果我可以想方设法让这个校验一直为true，那么就达到了绕过的目的
- 4网上找这个 securityCheck 方法定义是在哪，发现是native加密
- 5找到加载 so 文件的地方
- System.loadLibrary("crackme")  

![03](https://github.com/qqizai/CrackApp/tree/master/AliCrackme/statics/alicrackme03.jpg)
- 6直接去 lib 里面找到 so 文件
- 等于加载的so文件是： libcrackme.so  这个写法对应文件名关系就是：lib+加载的名字+.so  就是对应的 so 文件名

**到目前为止大概搞清楚了：其实这APP就是一个校验输入的密码是否跟APP内定的真正的密码完全一样，完全一样，那么就是直接进去对应的逻辑**

所以我们需要解决的问题就是：无论输入什么，都使这个if一直为true，这就达到我们的目的

开干吧！！！

**Just do IT!**

script.js:
```
//判断是否可用
if(Java.available){
    //frida固定写法Java.perform
    Java.perform(function(){
        //我们需要hook的类：MainActivity
        var MainActivity = Java.use("com.yaotong.crackme.MainActivity");
        
        //我们对这个securityCheck方法进行重新定义，无论输入什么都返回true
        MainActivity.securityCheck.implementation = function(args){
            var v = Java.androidVersion;
            send("version is " + v);
            send("received :" + args);
            send("hook successfully, NB!!");
            return true;
        };
    });
}
```

对应demo代码是：
```
#!/usr/bin/python3
# -*- coding: utf-8 -*-
# @Time     : 2020/11/13 0:22
# @Author   : qizai
# @File     : frida_hook.py
# @Software : PyCharm
import datetime
import sys

import frida


def on_message(message, data):
    if message["type"] == "send":
        print("[*] - {} {}".format(datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S.%f"), message["payload"]))
    else:
        print("[*] - {} message: {}".format(datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S.%f"), message))


def demo():
    """
    阿里官方的 demo——alicrackme
    用于学习 frida hook，主要是 hook 指定的函数，返回 true 即可绕过
    """
    app_name = "com.yaotong.crackme"
    jscode = open('script.js', 'r').read()
    process = frida.get_usb_device(timeout=10).attach(app_name)
    script = process.create_script(jscode)
    # 请注意，这个 "message" 是写死的，frida 里面是只能是这样写的
    script.on("message", on_message)
    print("[*] - {} Running CTF".format(datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S.%f")))
    script.load()
    sys.stdin.read()


if __name__ == "__main__":
    demo()
```

首先启动APP，然后再执行上面的程序，执行之后输出内容是：
```
[*] - 2020-11-22 12:06:55.850622 Running CTF
[*] - 2020-11-22 12:08:57.812562 version is 6.0
[*] - 2020-11-22 12:08:57.813559 received :我就问你谁
[*] - 2020-11-22 12:08:57.813559 hook successfully, NB!!
[*] - 2020-11-22 12:09:09.576112 version is 6.0
[*] - 2020-11-22 12:09:09.576112 received :我是qizai，感谢大家的阅读，一键三连走起
[*] - 2020-11-22 12:09:09.577110 hook successfully, NB!!
```

结果图：
![04](https://github.com/qqizai/CrackApp/tree/master/AliCrackme/statics/S01122-120827.jpg)
![05](https://github.com/qqizai/CrackApp/tree/master/AliCrackme/statics/S01122-120903.jpg)



#### 二、so动态调试



#### 三、本文小结


##### 额外建议

附件: AliCrackme.apk

```
链接: https://pan.baidu.com/s/1HEi3OMevTnTAWMxMZ8CaqA 
提取码: sas5 复制这段内容后打开百度网盘手机App，操作更方便哦
```



