---
layout:     post
title:      sms forward
date:       2023-04-29
author:     Admin
header-img: img/post/bg3.webp
catalog: true
tags:
    - 网海拾贝
---
&emsp;&emsp;1.拿到板子，插下面板子的Type-C口，启动luatools.exe，确保resource文件夹下有对应的固件LuatOS-SoC_V1005_ESP32C3_USB.soc
<br>
![test](https://img.locyoo.com/1148.png)
<br>
<div style="display:none">
&emsp;&emsp;2.打开https://push.luatos.org/，Github账号登录以后，直接建立消息通道，选择自定义post，参数如下，该自定义post是推送到企业微信机器人的，用的是群聊机器人的webhook，所以post格式必须符合腾讯的规范。
<br>
![test](https://img.locyoo.com/1149.png)
![test](https://img.locyoo.com/1150.png)
<br>
</div>
&emsp;&emsp;3.打开sms_forwarding-master\script\notify.lua，进行一下修改，输入WIFI的SSID和密码，修改luatospush的字符串，注意是建立了消息通道以后消息通道的字符串，而非首页显示的那个字符串。保存
<br>
![test](https://img.locyoo.com/1151.png)
<br>
&emsp;&emsp;4.勾选好以后点击项目测试，按下图顺序步骤操作，即可刷入成功
<br>
![test](https://img.locyoo.com/1152.png)
![test](https://img.locyoo.com/1153.png)
<br>
&emsp;&emsp;5.申请企业微信，建立我的企业，起个群名，在企业微信APP的群聊中，管理员申请好机器人，在群机器人的信息里，会有一个webhook地址，复制出来填到刚才自定义post，请求地址里。
<br>
&emsp;&emsp;测试无误后，大功告成！
<br>
&emsp;&emsp;上面的方法是使用wifi模块，如果用4G模块，第一步需要把Type-C插到上面这个板子，在luatools.exe中选择“4G模块USB打印”，而不是通用串口USB打印。再者在第三步中，需要修改和刷入的是sms_forwarding-master\only_Air780E_version\mian.lua，而非notify.lua，只修改一个串码就可以。最后在luatools.exe的项目测试中最好新创建一个项目，然后用LuatOS-SoC_V1107_EC618_FULL.soc这个底层core，选择目录也需要改为sms_forwarding-master\only_Air780E_version，把readme.md删掉，最后选择“下载底层和脚本”，按界面提示，摁住boot，再摁一下rst，最后松开rst，就可以看到正常刷入了。
<br>
![test](https://img.locyoo.com/1154.png)
<br>
&emsp;&emsp;刷好以后，看到这个界面就成功了！
<br>
![test](https://img.locyoo.com/1155.png)