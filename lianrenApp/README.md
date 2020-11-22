
先知：图片若看不了，请点击我去国内的码云查看：[https://gitee.com/qqizai/CrackApp](https://gitee.com/qqizai/CrackApp)

本文很长请注意喝水

想要了解最新动态，敬请关注[我的博客](https://blog.csdn.net/weixin_41173374)

更详细的内容，请看原文章来源：
[原文1](https://bbs.nightteam.cn/thread-428.htm#6219)
[原文2](https://qqizai.github.io/2020/05/04/app-lianren/)
[原文3](https://blog.csdn.net/weixin_41173374)


正文：

### 很荣幸大家能够找到这里来，这是我的APP逆向的处女作

[关于更多逆向知识，你可以来这里注册论坛账号，和各大优秀的博友们进行讨论](https://bbs.nightteam.cn/?uid=70)


**重点：先声明一下：本文仅仅是用于技术交流，而非传播sexy相关的东西，谢谢。**

这是一篇**入门级的APP逆向教程**，如有写着不对的地方，烦请大佬们指出，互相学习，谢谢啦


#### 前言

本文起源于：前两天，我的一位朋友跟我说，他在QQ群里面，一位群友，发了一些黑色/灰色地带的产业链信息，他就仅仅是好奇，去下载了那款APP

原来发现，QQ群友的分享，是为那款APP做引流，一个有效的分享可以获得多少金币(砖石)这种(是指引导别人来注册，才算有效分享，才能获取金币)

打开之后，发现很惊人，这搞的 带颜色 的直播啊！！！

而这直播APP里面区分：免费和付费的房间，当你想要看付费的房间，那你只能看前面几十秒钟的时间，过了这个时间就会弹窗出来，让你付费，才能继续看

 我朋友就让我帮忙看看，跟我说有什么能难倒程序员的？？而且刚好最近我在学APP逆向，心里想着，这种灰产的APP又不能大力宣传，估计反逆向不会做很好吧？然后就开始尝试搞搞，看看能不能逆了它 

##### 0x00、预估一下本次的逆向步骤

- [x] 第一步、抓包分析
- [x] 第二步、反编译
- [x] 第三步、根据第一步的抓包关键词来搜索
- [x] 第三步、根据抓包信息、收费弹窗，来分析这个收费的逻辑
- [x] 第四步、得出这个逻辑之后，对smali代码进行篡改，保存
- [x] 第五步、回编译APP，重新打包签名

> 本次用到的工具：魅族手机(Flyme6)已经root的、HttpCanary抓包工具、MT管理器、xxApp(这个不公开说了)


##### 0x01、首要就是抓包分析

由于本次使用的手机是root的，而且抓包工具的证书已经被我安装到系统证书的目录下，而非直接安装到用户个人证书(所以对于本次APP是否有检验抓包工具检测这一块，直接忽略掉)

打开HttpCanary抓包工具，然后找到【设置】

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200501221237276.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTE3MzM3NA==,size_16,color_FFFFFF,t_70#pic_center)

然后选择【目标应用】，这里为了过滤无用的包，减少干扰，利于快速分析

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200501221214445.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTE3MzM3NA==,size_16,color_FFFFFF,t_70#pic_center)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200501223012714.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTE3MzM3NA==,size_16,color_FFFFFF,t_70)

返回去之后，点击开始【抓包】，然后转向去打开目标APP，然后打开一些room，免费的、收费的都开一下，目的就是为了抓包，看看APP发起了哪些请求、收到哪些响应

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200501223421483.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTE3MzM3NA==,size_16,color_FFFFFF,t_70#pic_center)

再切换回来之后，发现已经存在一堆抓包信息了

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200501223044692.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTE3MzM3NA==,size_16,color_FFFFFF,t_70)

一个一个查看包响应，看看有什么收获

没想到，竟然发现如此大的突破口

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200501223109391.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTE3MzM3NA==,size_16,color_FFFFFF,t_70)

看下面的两张截图前，再次向广大博友们声明：本次教程仅仅是用于技术交流，而非传播sexy相关的，谢谢。

下面来看两张收费的room的就截图：一个用户点进去看了几十秒(应该是30s，具体是多少没有数)，然后就弹窗出来说得需要付费才能继续，这不诱惑人进行付费吗？

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200501223119958.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTE3MzM3NA==,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020050122312760.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTE3MzM3NA==,size_16,color_FFFFFF,t_70)

惹，真的是低俗呀，这都什么人啊？出卖自己body来挣钱？


##### 0x02、反编译APP

 1. 鄙人一开始是，使用 jadx-gui 来反编译APP，发现没有壳、没有很强的混淆，可以直接反编译

2. 反编译之后，进行相关的关键字来搜索，前面抓包中，有个很有用的关键词：

```
"type": "2",
"type_val": "38",
"type_msg": "本房间为收费房间，需支付38钻石"
```
再来简单分析一下：这三个字段中，如果你直接搜索 type 的话，根据经验，这肯定会出现很多搜索结果，排除它；而 type_val 、type_msg 这两个字段出现的概率不会很大，那么分析完了，就开始搜索呗！！！

首次我搜索的是 type_val ，但是出来的结果还是不够精准，有6/7条结果；
然后 转而去搜索 type_msg ，发现出来的结果只有两处，这不就是很精准了吗，噢耶！！

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200501230410779.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTE3MzM3NA==,size_16,color_FFFFFF,t_70#pic_center)

然后选择第二个进去看看。

你们可以思考一下为什么是选第二个呢？(我个人纯粹是靠大胆的猜测，因为第二个有关键词 LiveRoom，就是直播房间嘛)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200501231116935.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTE3MzM3NA==,size_16,color_FFFFFF,t_70#pic_center)

发现抓包中的三个参数，都在这里取出来了，而且发现是对那个 type 字段进行判断是否收费的，多次抓包分析得出返回的数据中：免费房间的 type=0，收费的房间 type=2，至于代码中还有一个 type=1 的判断，看到代码中调用的方法名，可以猜测：

 - type=0，调用 forwardNormalRoom 方法，根据字面意思是：转到正常(免费)房间处理逻辑
 - type=1，调用 forwardPwdRoom 方法，根据字面意思是：转到输入房间密码处理逻辑
 - type=2 || type=3，调用 forwardPayRoom 方法，根据字面意思是：转到房间收费的处理逻辑

 所以，我们只需要将，type=1/2/3里面的调用方法，改为调用type=0的方法即可绕过这个限制，话不多说，下面就是修改的步骤，Just do IT 

##### 0x03、修改 smali 代码，重新打包、签名

理清当前逻辑之后，本文使用最轻量级的、入门级的、小白级的工具—— MT管理器 

简单来简述一下它的功能：
- [x] 其他文件夹有的基础功能它也都有(复制、移动、重命名等等)
- [x]  APP安装包编辑 ，主要有 DEX 编辑，ARSC 编辑，XML 编辑，APK 签名、APK 优化、APK 共存、去除签名校验、RES 资源混淆、RES 反资源混淆、翻译模式等
- [x] 可以修改文件夹权限(系统目录的话，需要 root 权限才能修改)
- [x] 自带强大的文本编辑器，可以流畅编辑大文本文件，支持设置是否显示行号、开关自动换行、双指缩放字体大小、自动识别编码、代码语法高亮、自动缩进、正则搜索替换

其他更加强大功能，待你们去发掘，相信用过了之后，你们会觉得这是用来修改APP包/破解APP限制的一个好工具。

本次主要是使用它的第二个功能—— APP安装包编辑功能 

 1. 使用MT管理器【查看】目标APP包
 
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200502211014480.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTE3MzM3NA==,size_16,color_FFFFFF,t_70#pic_center)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200502202144885.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTE3MzM3NA==,size_16,color_FFFFFF,t_70)
 
图中我圈了红色的三个文件是我们待会需要进行操作的

其中，第一个 META-INF 目录中，包含了一些签名文件等，我们将里面的 三个文件删除掉：CERT.RSA、CERT.SF、MANIFEST.MF（其中CERT.RSA、CERT.SF这两个命名，不同APP可能会名字不一样，只要找到后缀格式一样的即可）。

Tips：鄙人在这里踩了一个坑，这个好像得需要删掉这三个文件打包才不会被检测到。不然重新打包签名之后，一联网打开APP就会出现 闪退 的情况，估计是被检测检验了。

文件  classes.dex、classes2.dex  就是APP编译之后，Class转成成.dex文件，dex是Android平台上(Dalvik虚拟机)的可执行文件, 相当于Windows平台中的exe文件, 每个Apk安装包中都有dex文件, 里面包含了该app的所有源码, 通过反编译工具可以获取到相应的java源码。

一些工程很大的话就会产生多个.dex文件，到现在有了前面的介绍，我们可以简单的知道：dex 文件就变相等于可执行的源码，那我们可以通过修改这个源码来达到逆向的目的(说法如有说错，烦请大佬们纠正，谢谢啦)。

 2. 先打开第一个dex文件，MT管理器会提示我们，要用哪个打开，选择 【Dex编辑器++】即可
 
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200502211149489.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTE3MzM3NA==,size_16,color_FFFFFF,t_70#pic_center)

3.选择【搜索】 ，然后输入刚刚那些关键词【type_msg】

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200502223039562.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTE3MzM3NA==,size_16,color_FFFFFF,t_70#pic_center)


4. 点击右上角，再次进行搜索：【type_msg】

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200502212148693.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTE3MzM3NA==,size_16,color_FFFFFF,t_70#pic_center)

此处，你得需要一些smali语法，但是本文我只介绍本文使用的语法：
- [x] if-eqz : 意思就是 是否等于0（if equal zero）
- [x] if-eq：意思就是 是否相等 （if equal）
- [x] const/4 p2，0x1：意思就是将0x1，即0x01赋值到 p2 ，即 p2=1

所以，```if-eqz p1，：cond_4b```就是：如果 p1=0，那么调用方法 cond_4b;
```if-eq p1,p2，：cond_45```如果 p1=p2(p1=1)，那么调用方法 cond_45 ；

这也印证了我们前面反编译APP的源码逻辑。
那肯定会有小伙伴会问，那个 cond_4b 是什么方法？很好，我这里回答一下，因为通过我们前面反编译的分析，我们已经知道，如果type=0的时候就是调用 forwardNormalRoom 方法，所以，可以肯定的告诉你们，这个cond_4b就是指向这个 forwardNormalRoom 方法。

5. 将type=1/2/3的情况，调用的方法改为调用 cond_4b 即可，然后点击右上角的保存

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020050221461558.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTE3MzM3NA==,size_16,color_FFFFFF,t_70#pic_center)

6. 保存之后，返回上一层第一层搜索的时候，点击右上角，选择【编译】

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200502220250206.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTE3MzM3NA==,size_16,color_FFFFFF,t_70#pic_center)

7. 编译之后进行返回，返回的时候会提示你，点击【退出】即可。此时会再次提示你，dex文件已经修改，问你是否更新到压缩包中 ，我们选择【确定】

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020050222080531.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTE3MzM3NA==,size_16,color_FFFFFF,t_70#pic_center)

然后你会看到压缩在更新

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200502220949175.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTE3MzM3NA==,size_16,color_FFFFFF,t_70#pic_center)

8. 然后返回之后，你会看到一个刚刚被修改过的apk文件，显示绿色名字的，然后点击它，第一步选择【功能】、第二步选择【APK签名】

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200502221547811.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTE3MzM3NA==,size_16,color_FFFFFF,t_70#pic_center)

选择【确定】即可

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020050222182185.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTE3MzM3NA==,size_16,color_FFFFFF,t_70#pic_center)

然后你就可以【猥琐欲为】啦

##### 0x04、最后重新签名之后，你可以卸载原来的APP，然后装上这个无限制的版本，哈哈哈... 

**本文 入门级的APP逆向教程 已经编写完毕，妈呀，花了几个小时来重新演示逆向过程，以及编写本篇文章**

##### 最后: 真感谢每一位小伙伴，竟然看完了我的文章，谢谢你们啦，有帮助的话，点个赞呀

题外话：希望小伙伴们都别用于非法的手段，本文只是技术交流

APK：
```
链接: https://pan.baidu.com/s/1g4VU_fAEX1szseFB9dwfNg 
提取码: w9xs 复制这段内容后打开百度网盘手机App，操作更方便哦
```

**谢谢大家**

相关文献参考：
> [META-INF目录是干啥用的？](https://blog.csdn.net/qq_38449518/article/details/82414069?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-1&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-1)


##### 赞赏

如果你觉得笔者辛苦了，可以的话请我喝杯咖啡，感谢你的支持

![zanshangma](./statics/zanshangma.png)

你的赞赏就是我的动力

