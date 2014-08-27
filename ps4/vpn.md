------------------教程完全转自boafanx----------
用一个笔记本电脑就可以给ps4做代理了，因为客户端比较稳定，而且电脑的解码能力比较强大，所以效果上要胜于路由器端做代理。


--------------教程原文----------------



【写在前面】
Shadowsocks为当下最强代理，没有之一。因为它是多线程的，而且无需时刻与服务器保持连接状态，所以效果秒杀ssh和vpn。顺便说一下vpn，是目前最低级的代理，不灵活、无tcp加速、成本高等缺点比较严重，不建议大家使用。
本文照顾小白群体，详细介绍了如何让ps4使用到强大的shadowsocks代理。
为了方便大家，我做好了一站式傻瓜包。我已经做好了关键设置，你下载下来以后，只需要解压、填入shadowsocks账号信息、运行软件就可以轻松搭建好本地代理服务器！


【教程开始】

1.下载一站式傻瓜包：http://boafanx.tabboa.com/usr/uploads/2014/05/572436193.zip


2.解压。（求你败直接在压缩包里运行，那样不科学...）


3.重点，用记事本打开config.json，填写好shadowsocks账号信息，保存。
这个账号信息在boafanx官网有免费提供，地址：http://boafanx.tabboa.com/boafanx-ss/
如果觉得免费账号不爽，可以去买收费的，建议电信用户买新加坡、香港的，联通用户买日本的。因为不同网络、地区的表现个不同，所以要测试好了再买，否则买来不一定能用。


4.重点，运行boafanx-ss客户端.exe。
随后系统托盘（屏幕右下角）会有运行提示信息，如果你点开图标，会看到类似于下图的命令窗口，说明shadowsocks代理正常运行中，再次点击
![ico](http://imgsrc.baidu.com/forum/w%3D580/sign=542e8bbf4936acaf59e096f44cd88d03/54743ac79f3df8dc4ec14717cf11728b451028f7.jpg)
图标就会关闭命令窗口。
![](http://imgsrc.baidu.com/forum/w%3D580/sign=0e6ed36ba1cc7cd9fa2d34d109002104/dc67d000baa1cd11d04abaa6bb12c8fcc1ce2df7.jpg)


5.重点，运行privoxy.exe。
随后会出现空白窗口，可以直接关闭/最小化，这个小程序的作用就是把shadowsocks代理转换为ps4、psv、ipad、iphone5s等设备所支持的http代理。


6.此时代理已经做好了，Proxy服务器地址是你电脑的内网IP，端口固定是8118 

这个代理叫做“http代理”，这种代理的普及度最广，所以索尼和苹果的设备也支持。顺便说一下，ie、360、chrome等浏览器直接支持，如果你不会设置浏览器代理，请尝试百度搜索“ie设置代理服务器”等关键词。


重要：不知道内网IP的话，可以在cmd里（视窗键
![](http://imgsrc.baidu.com/forum/w%3D580/sign=d6c8b680aeaf2eddd4f149e1bd110102/1f7c9e3df8dcd1001a6030a7708b4710bb122ff7.jpg)
图片来自：lulu_iboa的百度相册
+R，输入cmd，然后回车），使用ipconfig命令查看，IP Address 或 IPv4地址就是我们需要的内网IP，如下图所示：
![](http://imgsrc.baidu.com/forum/w%3D580/sign=8d1c5e7a7ed98d1076d40c39113eb807/f5bbbba1cd11728be4d144a4cafcc3cec1fd2cf7.jpg)


对于我来说，http代理地址就是192.168.199.111，端口为8118
那么，在游戏机里怎么连呢？很简单，我以psv为例，详细说明一下。


【设定】-【网络】-【Wi-Fi设定】，戳进wifi连接，到【详细设定】里面，会看到【Proxy服务器】，选择【使用】，然后输入刚才查到的http代理地址192.168.199.111，和端口8118，用户名密码不要填。这样，代理就设置好了！


怎么知道代理是否生效了呢？很简单，打开psv的网页浏览器，输入www.facebook.com，如果能打开则说明代理好用！


同理，ipad、iphone等ios设备可以在【设置】-【无线局域网】，戳进wifi连接，拉倒最下面，在HTTP代理【手动】里面填上刚才查到的http代理地址192.168.199.111和端口8118就可以了，【鉴定】那里不要管。


站长亲测，psv使用日本代理以后，下载速度飙升（使用代理前下载完成需要数百分钟，使用代理后仅需要几分钟），还可以fq出去看youtube视频。



> Written with [StackEdit](https://stackedit.io/).