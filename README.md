本人计算机能力一般，只是在已有的工作上做了简单的修改，主要参考了[每日自动体温上报](https://mp.weixin.qq.com/s/m1Ie4gftS79BdtaMp-4eTg)、["平安报"自动化折腾日记](https://blog.vincenteliang.cn/2020/05/31/ustb_checkIn_auto#%E6%8A%93%E5%8C%85-packet-capture)、[USTB-daily-report](https://github.com/Jason23347/USTB-daily-report)。文章[每日自动体温上报](https://mp.weixin.qq.com/s/m1Ie4gftS79BdtaMp-4eTg)提供了自动上报的操作思路，["平安报"自动化折腾日记](https://blog.vincenteliang.cn/2020/05/31/ustb_checkIn_auto#%E6%8A%93%E5%8C%85-packet-capture)进行了具体实现，利用Charles软件抓包，使用shell编程实现了自动填报的功能，代码在GitHub仓库[USTB-daily-report](https://github.com/Jason23347/USTB-daily-report)。我把shell代码改成了python，并加入了微信推送的功能，仓库地址为[https://github.com/hanfangyuan4396/USTB-daily-report](https://github.com/hanfangyuan4396/USTB-daily-report)。另外，在上报的数据中，有个位置信息是根据手机的GPS生成，是学校判断学生位置的重要依据，但使用自动上报的方式，这个位置信息是可以自由编辑的，还请大家自觉遵守，如实上报位置。本教程目的是帮大家免去每天提交健康信息的重复任务，但同时也需要大家自觉保证上报的信息真实。不过有位博士借助home assistance软件自动定位用户的位置并上传，有兴趣的同学可以自行研究实现[ustb-daily-with-hass](https://github.com/terrance-liang/ustb-daily-with-hass)(什么叫博士的严谨啊，战术后仰)。废话到此，下面进入正题。

# 1. 抓取填报信息

要想实现自动上报，首先需要获取到每天上传报的信息，我们需要获取两部分内容，一部分叫sessionid，相当于我们储存在计算机中的身份证号，计算机靠它来判断上报数据的人是谁，另一个部分就是每天上报的信息，比如你的体温、地址等信息，下面就介绍一下如何获取这两部分信息。

## 1.1 iOS用户

iOS用户相对来说获取比较简单，借助stream软件即可实现上述功能。

### 1.1.1 下载Stream

在AppStore中搜索下载Stream软件。
![在这里插入图片描述](https://img-blog.csdnimg.cn/9e4a53b05ccc449cae3767264f88bc80.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5rC055S16LS55ZiO5ZiO,size_20,color_FFFFFF,t_70,g_se,x_16)

### 1.1.2 安装证书

进入stream软件，点击开始抓包，会提示VPN配置信息，点击**允许**。
![在这里插入图片描述](https://img-blog.csdnimg.cn/ec8dcd643ac4477282d7606653675b47.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5rC055S16LS55ZiO5ZiO,size_20,color_FFFFFF,t_70,g_se,x_16)
接着会弹出安装证书提示，点击**去安装证书**。
![在这里插入图片描述](https://img-blog.csdnimg.cn/e847f3b1806146778c7df443bca354c4.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5rC055S16LS55ZiO5ZiO,size_20,color_FFFFFF,t_70,g_se,x_16)
点击**步骤一:安装CA证书**。
![在这里插入图片描述](https://img-blog.csdnimg.cn/23af3fdd16d74a69bb9e5a70d80ee2a4.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5rC055S16LS55ZiO5ZiO,size_20,color_FFFFFF,t_70,g_se,x_16)
下载配置描述文件，点击**允许**。
![在这里插入图片描述](https://img-blog.csdnimg.cn/aa2b17771f174058ab85037d568c4ac3.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5rC055S16LS55ZiO5ZiO,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/0ac6d7225f3342c3b4ca3473aeea0e26.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5rC055S16LS55ZiO5ZiO,size_20,color_FFFFFF,t_70,g_se,x_16)
下载完成后，进入手机的设置页面，点击**已下载描述文件**。
![在这里插入图片描述](https://img-blog.csdnimg.cn/fa8824ec0299409ca05eadaeace06a66.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5rC055S16LS55ZiO5ZiO,size_20,color_FFFFFF,t_70,g_se,x_16)
点击右上角的**安装**。
![在这里插入图片描述](https://img-blog.csdnimg.cn/5901d4dab03943ef93ec73652e132bee.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5rC055S16LS55ZiO5ZiO,size_20,color_FFFFFF,t_70,g_se,x_16)

![在这里插入图片描述](https://img-blog.csdnimg.cn/eeeb4549214f40b1af3d749dfcc9cbbd.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5rC055S16LS55ZiO5ZiO,size_20,color_FFFFFF,t_70,g_se,x_16)
安装完成。
![在这里插入图片描述](https://img-blog.csdnimg.cn/d3bb27e6ec314f61a18eb93ef9d37267.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5rC055S16LS55ZiO5ZiO,size_20,color_FFFFFF,t_70,g_se,x_16)
进入手机设置->通用->关于本机，拉到最底部，进入**证书信任设置**，信任刚刚安装的证书。至此，证书安装完毕。

![在这里插入图片描述](https://img-blog.csdnimg.cn/9713b63ad57a40c3bfd9d981f996bd3a.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5rC055S16LS55ZiO5ZiO,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/78064e28209f474c9cbe089c952f4f99.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5rC055S16LS55ZiO5ZiO,size_20,color_FFFFFF,t_70,g_se,x_16)

### 1.1.3 抓取上报信息

安装并信任证书后，回到stream软件，回提示让我们信任证书，不过我们已经信任证书了，直接点击**我已经信任过了**。
![在这里插入图片描述](https://img-blog.csdnimg.cn/68ab46d245a44e309d16dbe0b1ac1b7c.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5rC055S16LS55ZiO5ZiO,size_20,color_FFFFFF,t_70,g_se,x_16)
点击**设置抓包模式**，
![在这里插入图片描述](https://img-blog.csdnimg.cn/eedc44f6cd344ae79caeb736724def0d.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5rC055S16LS55ZiO5ZiO,size_20,color_FFFFFF,t_70,g_se,x_16)
开启**白名单模式**
![在这里插入图片描述](https://img-blog.csdnimg.cn/d32ca1b2098246f491987704ca575894.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5rC055S16LS55ZiO5ZiO,size_20,color_FFFFFF,t_70,g_se,x_16)
设置抓包的域名为 *.ustb.edu.cn，然后点击立即生效
![在这里插入图片描述](https://img-blog.csdnimg.cn/fc78e7298d1f429f9a5009c63a915f9b.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5rC055S16LS55ZiO5ZiO,size_20,color_FFFFFF,t_70,g_se,x_16)
然后点击开始抓包，回到微信提交平安报
![在这里插入图片描述](https://img-blog.csdnimg.cn/8d7d21d96d9d4c7ba7829c8800692a77.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5rC055S16LS55ZiO5ZiO,size_20,color_FFFFFF,t_70,g_se,x_16)
提交后，回到stream软件，点击请求数查看抓到的包，点击POST开头的请求
![在这里插入图片描述](https://img-blog.csdnimg.cn/fe32d42cd970450aaed6c5a8cad50814.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5rC055S16LS55ZiO5ZiO,size_20,color_FFFFFF,t_70,g_se,x_16)

查看POST请求，在请求头部的最下面可以看到JSESSIONID=XX
![在这里插入图片描述](https://img-blog.csdnimg.cn/c14ee24dd1144057aba3ce49e0df0d40.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5rC055S16LS55ZiO5ZiO,size_20,color_FFFFFF,t_70,g_se,x_16)

向下滑动，点击查看表单
![在这里插入图片描述](https://img-blog.csdnimg.cn/f2872b1e05dd455ba7b0d66429f92151.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5rC055S16LS55ZiO5ZiO,size_20,color_FFFFFF,t_70,g_se,x_16)

可以看到如下信息，即为上报的信息
![在这里插入图片描述](https://img-blog.csdnimg.cn/a306c4d2776045c0940b6d5c551ef15c.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5rC055S16LS55ZiO5ZiO,size_20,color_FFFFFF,t_70,g_se,x_16)
复制JSESSIONID和上报的信息，至此抓包完成。