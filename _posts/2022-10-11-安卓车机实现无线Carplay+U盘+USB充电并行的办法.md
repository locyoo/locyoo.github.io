---
layout:     post
title:      安卓车机实现无线Carplay+U盘+USB充电并行的办法
date:       2022-10-11
author:     Admin
header-img: img/post/bg3.webp
catalog: true
tags:
    - 爱折腾
---
&emsp;&emsp;我的车机是几年前装的飞歌的，只有一个USB插口，前几天想用Carplay，但是又想把装满无损歌曲的U盘插上，还有给行车记录仪USB供电，所以就有了这篇文章。
<br>
&emsp;&emsp;一、实现Carplay：往上买个靠谱的Carplay盒子，我买的这家分有线和无线两种，我买的无线的。无线Carplay的盒子买回来需要搭配车机上的APP应用操作，iPhone蓝牙连接到Carplay盒子即可。
<br>
![test](https://img.locyoo.com/1018.jpg)
<br>
&emsp;&emsp;该模式要求车机至少要支持USB2.0 Hi-Speed、支持USB Host Mode。
<br>
&emsp;&emsp;二、插U盘这个问题怎么解决？因为U盘需要的USB模式是车载设备USB是USB Device Mode。所以需要一根线，一分二，并且同时支持USB host和USB device两种模式。于是我买个一根这个。
<br>
![test](https://img.locyoo.com/1019.jpg)
<br>
&emsp;&emsp;三、上面两个问题都解决了，现在行车记录仪的USB供电问题怎么解决？拖出来查到点烟器这儿太乱，接车的保险处取电我嫌麻烦。于是我又买了一根线。这根线是一分三，把上面那根线本来应该插U盘的这个口，分出来三个，三个里面有一个支持读取U盘，有两个支持供电。所以现在行车记录仪的USB取电在这个一分三的USB转接器上。
<br>
![test](https://img.locyoo.com/1020.jpg)
<br>
&emsp;&emsp;全部设置好以后开机一试，无线Carplay、用车机自带的播放器读取U盘听歌、行车记录仪正常工作，都没有问题。
<br>
&emsp;&emsp;如果是有两根USB供电线的车机，那就更好办了。