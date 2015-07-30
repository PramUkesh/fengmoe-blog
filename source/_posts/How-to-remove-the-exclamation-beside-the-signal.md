title: Android 5.0 网络图标感叹号解决方法
date: 2015-07-30 18:15:32
tags:
- 转载
- Android 5.0
- 实用技巧
---

> 文章转载自：ACG喵（http://www.acgmiao.com/）

> 原文地址：http://acgmiao.com/posts/portal_detection/

------

2014 年 10月，Google 发布了 Android 5.0 操作系统，新系统有着很多令人称赞的新功能，相信不少朋友会尝试去升级更新。

<table>
<tr>
<th>
<img src="http://acgmiao.com/wp-content/uploads/2015/07/portal_detection_pic5-576x1024.png" width="100%"/>
</th>
<th>
<img src="http://acgmiao.com/wp-content/uploads/2015/07/portal_detection_pic4-576x1024.png" width="100%"/>
</th>
</tr>
</table>


Google 在新系统中加入了网络的检测机制，在原生系统当中，它通过连接 Google 服务器来测试网络连通状况，如果无法联通，会在网络中提示受限，显示为网络图标右下角为小感叹号。

这种设计的用意在于如果它发现 WiFi 网络受限或无连接，而且设备有移动网络可用，可以自动切换到2G/3G/LTE，让设备保持联网。

不过由于你懂得原因，Google 服务器无法访问，这就会导致提示网络受限，而且因为不断地向网络发包，还会消耗流量和影响手机续航时间。

### 解决方法

先下载ADB调试工具，安装手机驱动，打开开发者选项中的“USB调试”开关。

1. 完全屏蔽网络检查功能，最简单快速，但是就没有办法提示 WiFi 登录

 `adb shell "settings put global captive_portal_detection_enabled 0"`

2. 用国内的服务器替换掉 Google 的服务器：

 `adb shell "settings put global captive_portal_server portaldetection.sinaapp.com"`

3.  （此方法需要Root权限）访问/data/data/com.android.providers.settings/databases/settings.db用sqlite编辑器打开，修改表globle内容。

4. （此方法需要Root权限）使用 Portal Server 修改器进行修改。[下载地址](http://www.noisyfox.cn/download/75/)

<table>
<tr>
<th>
<img src="http://acgmiao.com/wp-content/uploads/2015/07/portal_detection_pic6-576x1024.png" width="100%"/>
</th>
<th>
<img src="http://acgmiao.com/wp-content/uploads/2015/07/portal_detection_pic3-576x1024.png" width="100%"/>
</th>
</tr>
</table>

#### 其他服务器

如果你对我提供的服务器速度不满意，你还可以尝试以下服务器地址：

```
xn--yet824cpd.xn--fiqs8s
liukebin.sinaapp.com
www.iwch.me
www.265.com
g.cn
tools.lnter.tk
clients3.x.gg
5.0.appinn.com
```

以上网址网络收集得到，如果给提供者带来不便之处请回复告知，我会尽快删除。

### 如何搭建一个自己的检测服务器

你看你就是一个爱折腾的人，也叫极(sha)客(zi)。

1. Apache服务器，如果你的服务器安装了rewrite模块，那么只需要在网站的.htaccess中加入以下代码：

 ```
 <IfModule mod_rewrite.c>
    RewriteEngine On
    RewriteCond %{REQUEST_URI} /generate_204$
    RewriteRule $ / [R=204]
 </IfModule>
 ```

2. nginx服务器，在配置文件中填入以下代码：

 `location /generate_204 { return 204; }`

3. SAE平台，在config.yaml中添加以下代码：

 ```
 handle:
  - rewrite:if (%{REQUEST_URI} ~ "/generate_204$") goto "$/ [R=204]"
 ```

4. 强行空文件法

直接在网站的根目录下建立一个叫做 generate_204 的空文件即可，因为如果返回的内容为空那么也会当成 204 来处理。

### 根据国内开发者[小狐狸](https://www.noisyfox.cn/)的解释，该机制运行方式如下：

![](http://acgmiao.com/wp-content/uploads/2015/04/portal_detection_pic1.png)

简要来说就是，如果该网络是 VPN ，那么直接使用这个网络进行连接，否则调用 **isCaptivePortal()** 函数进行网络状况的判定，再根据判定结果决定是否选用此网络。 而罪魁祸首就是这个 **isCaptivePortal()** 函数，它会访问 __clients3.google.com/generate_204__ 并根据返回结果来判断网络联通状况。

想必大家都连接过那些需要验证才能使用的 WiFi 热点吧，当你们连接这些热点的时候，Android 会自动弹出提示询问你是否需要登录。而这个功能就是依靠了 **isCaptivePortal()** 这个函数才得以实现，具体原理如下：

![](http://acgmiao.com/wp-content/uploads/2015/04/portal_detection_pic2.png)

如果当前 WiFi 是需要登录才可以连接，那么当试图访问 Google 的服务器的时候，WiFi 的验证机制一定会自动跳转到一个登录页面，这个时候 HTTP 请求的返回值就必然不是 204。就是通过这一机制，便可以区分是否需要验证。


###### 文章转载自 [ACG喵](http://acgmiao.com)：http://acgmiao.com/posts/portal_detection/
