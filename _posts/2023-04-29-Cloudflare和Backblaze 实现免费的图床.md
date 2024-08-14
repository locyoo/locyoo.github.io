---
layout:     post
title:      本文已失效！！Cloudflare和Backblaze 实现免费的图床
date:       2023-04-29
author:     Admin
header-img: img/post/bg3.webp
catalog: true
tags:
    - 网海拾贝
---
本文已失效！！
<br>
&emsp;&emsp;Backblaze 提供的存储服务，B2 云存储提供 10 GB 的免费空间，同时 Cloudflare 与 Backblaze 之间的流量不计费，用作为图床是完全足够了。
<br>
&emsp;&emsp;使用 Backblaze B2 作为图床的唯一要求就是拥有一条托管在 Cloudflare 上的域名。
<br>
# 创建桶
<br>
&emsp;&emsp;打开 Backblaze 官网很容易就能找到 B2 Cloud Storage 产品，完成注册与邮箱验证后，登录即可免费创建 B2 云存储的桶。
<br>
![test](https://img.locyoo.com/1177.png)
<br>
![test](https://img.locyoo.com/1178.png)
<br>
&emsp;&emsp;选择 Create a Bucket，在 Bucket Unique Name 一栏填入桶名称，桶名决定了源站的 URL，应尽可能复杂避免被他人猜测到。若源站 URL 泄露，绕过 Cloudflare 的直接访问就会产生额外流量了。其余项如下图保持默认即可：
<br>
![test](https://img.locyoo.com/1179.png)
<br>
&emsp;&emsp;创建完成后，选择 Upload / Download 尝试在桶中上传一张图片，查看图片的详细信息，其中 Friendly URL 一项就是生成的图片链接。
<br>
![test](https://img.locyoo.com/1180.png)
<br>
&emsp;&emsp;以 f000.backblazeb2.com/file/a-complicated-name/hokciu.jpg 为例，图片链接可以都分成以下几个部分：
<br>
![test](https://img.locyoo.com/1181.png)
<br>
&emsp;&emsp;因为 Friendly URL 中包含了桶名，不宜直接引用。假设想要将链接改写为 img.leonis.cc/hokciu.jpg，显然要修改主机名、隐藏固定的后缀和桶名，再拼接上图片路径，URL 的改写就通过 Cloudflare 实现。
<br>
# 添加 DNS 记录
<br>
&emsp;&emsp;改写的目标 URL 必须使用 Cloudflare CDN，打开 Cloudflare 控制台，添加名称为 img 目标为 f000.backblazeb2.com 的 CNAME 记录，并将代理状态设为打开。待 DNS 记录生效后，就实现了 img.leonis.cc → f000.backblazeb2.com 的跳转。
<br>
![test](https://img.locyoo.com/1182.png)
<br>
# 配置转换规则
<br>
&emsp;&emsp;同样在 Cloudflare 控制台中，找到 规则 - 转换规则 页面并创建新规则，填写规则自定义名称后就来处理 URL 的转换问题。
<br>
&emsp;&emsp;第一次接触 Cloudflare 的转换规则功能时，我被界面上各个选项弄得很迷糊，所以我在这里介绍一下转换规则各个功能的使用方法，读者理解了就能根据自己的想法配置图片链接了。
<br>
&emsp;&emsp;规则页面上的「传入请求」是指访客对托管站点发起的请求，例如访客所浏览的页面上有一条 img.leonis.cc/hokciu.jpg 链接，该请求先进入到 Cloudflare 的服务器，再根据设定的规则前往 f000.backblazeb2.com/file/a-complicated-name/hokciu.jpg 取出图片资源，最终呈现在页面上。
<br>
&emsp;&emsp;前文为了表述简单，说的是将 f000.backblazeb2.com/* 改为 img.leonis.cc/*，实则是我们要设定一个规则，让访客能通过 img.leonis.cc/* 到 f000.backblazeb2.com/* 中取得需要的图片。
<br>
&emsp;&emsp;在规则页面中的设置项可以参考下图：
<br>
![test](https://img.locyoo.com/1183.png)
<br>
&emsp;&emsp;该规则筛选得到所有主机名为 img.leonis.cc 的请求，将其 URL 重写到 concat("/file/a-complicated-name", http.request.uri.path)，也就是把所有对 img.leonis.cc/* 的请求指向 img.leonis.cc/file/a-complicated-name/*。而因为 img.leonis.cc 已经通过 CNAME 指向了 f000.backblazeb2.com，最终请求都到达 f000.backblazeb2.com/file/a-complicated-name/* 并取得图片资源。
<br>
&emsp;&emsp;上述请求过程可以表示成：
<br>
![test](https://img.locyoo.com/1184.png)
<br>
&emsp;&emsp;需要注意的是，因为这里使用的是重写（rewrite）而非重定向（redirect），请求的改变发生在服务端而非客户端，整个过程中用户都不会看见 URL 发生变化，所以也就达到了隐藏桶名的目的。
<br>
&emsp;&emsp;若设置全部无误，这时候就可以通过 https://img.leonis.cc/example.jpg 打开先前上传的图片了，由于 Backblaze 只支持 HTTPS，若打开 http://img.leonis.cc/example.jpg 则会弹出无效页面，用户体验不太好，所以接下来我们还需要通过 Cloudflare 页面规则完成 HTTPS 重写和缓存的相关设置。
<br>
# 设置页面规则
<br>
&emsp;&emsp;回到 Backblaze 找到 Bucket Settings 一项，在 Bucket Info 中填入 {"cache-control":"max-age=720000"}，该项将 Cloudflare 回到源站获取资源的周期设定为 720000 s，用于避免回源次数过多导致加载速度过慢。当然，该周期过长也会导致源文件更改后不能及时更新，可以按自己的需求更改。
<br>
![test](https://img.locyoo.com/1185.png)
<br>
&emsp;&emsp;在 Cloudflare 中打开 规则 - 页面规则，新建一条页面规则，在 URL 一栏中填入 img.leonis.cc/*，按下图设置设置缓存和 HTTPS 即可。
<br>
![test](https://img.locyoo.com/1186.png)
<br>
&emsp;&emsp;暂时不确定边缘缓存 TTL 和缓存级别两个设置项有什么作用，发现在未设置时图片就能命中缓存。不过既然官方文档提到了这两项配置就先给开启了，回头找找有没有详细些的资料。
<br>
&emsp;&emsp;再打开样例图片的链接，查看浏览器的开发者工具，在响应头中有一项 cd-cache-status，其值若为 HIT，则表示 Cloudflare 命中了缓存，该图片是由缓存中取出的。
<br>
![test](https://img.locyoo.com/1187.png)
<br>
&emsp;&emsp;至此关于 Backblaze + Cloudflare 的图床就设置完了，接下来还可以借助 PicGo 等第三方工具更方便地上传图片并获取图片链接，这部分内容可以根据章节标题向后文寻找。
<br>
# 整合静态资源
<br>
&emsp;&emsp;由于博客通常会使用到包括图片、字体在内的多种静态资源，我希望将他们都整合到相同的子域名下。当某些静态资源由于各种原由突然挂掉的时候（说的就是 jsDelivr 和 Google Fonts），我就可以直接在 Cloudflare 控制台上将其指向备用服务而不用去网页中一个个修改引用的链接，在管理维护上更方便。如果读者没有此需求，就可以完整跳过这一节了。
<br>
![test](https://img.locyoo.com/1188.png)
<br>
&emsp;&emsp;在我的设想中，所有静态资源都由 cdn.leonis.cc 分发，通过 URL 路径转向不同的子域名取得目标资源，后面就以图片资源为例实现这个构想。
<br>
## 添加 CDN 子域名
<br>
&emsp;&emsp;先在 Cloudflare 中添加子域名 cdn.leonis.cc 的 DNS 记录，暂时任意设置一个解析目标，能让 Cloudflare 获取缓存即可。
<br>
## 处理 URL 重定向
<br>
&emsp;&emsp;接着要实现对 URL 路径的处理，例如将 cdn.leonis.cc/img/* 重定向到 img.leonis.cc/*，这种重定向可以通过 Cloudflare 规则功能下的页面规则或重定向规则实现。
<br>
&emsp;&emsp;若使用页面规则，可以使用下图中的方案，用通配符实现 URL 解析：
<br>
![test](https://img.locyoo.com/1189.png)
<br>
&emsp;&emsp;该方案的一个小缺点在于无法将规则应用于 cdn.leonis.cc/img 等不带后一个 / 的页面。使用重定向规则可以解决这个问题，但重定向规则中的正则匹配是收费功能，无法批量处理，每种后缀都必须添加一条规则，配置方案可以参考下图：
<br>
![test](https://img.locyoo.com/1190.png)
<br>
&emsp;&emsp;表达式 concat("https://img.leonis.cc", substring(http.request.uri.path, 4)) 中的 substring() 用于除去 /img/* 的前 4 个字符，若是用于处理 /js/* 等不同的 URL 则需要根据字符数量更改该数值。以上两种方案各有优劣，读者可以根据自己的需求选择。
<br>
## 设置防盗链
<br>
&emsp;&emsp;防盗链是用于屏蔽其他站点对静态资源引用的常用手段，倒不是不愿意分享资源，至少本站内的各种照片都可随意使用，而是个人站点的服务容量有限，很难做到再向外提供服务。除此以外，设置防盗链对于避免流量被恶意浪费也很有必要。防盗链的功能可以通过 Cloudflare 的防火墙规则实现，打开 安全性 - WAF 页面即可创建规则。
<br>
&emsp;&emsp;防盗链功能一般通过请求头中的 Referer 字段判断是否允许请求，例如允许自己的站点引用图片（Referer 为本站 leonis.cc），不允许他人的站点引用图片（Referer 为外站 bing.com）。另外还有一种没有 Referer 的情况，例如直接打开图片、在各种 Markdown 编辑器中使用图片都属于这一类。
<br>
&emsp;&emsp;为了不影响正常使用，我使用的防盗链规则是允许无 Referer 与白名单站点访问。很棘手的是，Cloudflare 没有提供判断有无 Referer 的功能，所以我使用了比较曲折的方法实现该方案。
<br>
&emsp;&emsp;首先新建一条防火墙规则，对于静态资源的 URL，阻止所有 Referer 中包含 "http" 的请求：
<br>
![test](https://img.locyoo.com/1191.png)
<br>
&emsp;&emsp;该规则实际上阻止了所有具有 Referer 的请求，由于无法使用通配符才用 "http" 作为匹配内容。需要注意的是，没有 Referer 的请求不在该匹配范围内，设置后仍可访问。
<br>
&emsp;&emsp;再新建一条规则，这条规则用于根据 Referer 放行请求，作用等同于白名单，设置项如下：
<br>
![test](https://img.locyoo.com/1192.png)
<br>
&emsp;&emsp;设置生效后可以发现，先前的图片链接可以直接打开，却不能在其他网站上引用了。Cloudflare 阻止了白名单以外站点的引用请求，在防火墙事件中还可以查看阻止请求的来源 IP 等具体信息。
<br>
![test](https://img.locyoo.com/1193.png)
<br>
&emsp;&emsp;后来发现在 Cloudflare 控制台中的 Scrape Shield 页面中有一项 Hotlink 保护功能，一键即可开启防盗链，在 Configuration Rules 中添加规则即为白名单，该配置方案更简单，以上 WAF 方案也留作参考。
<br>
# PicGo 设置
<br>
&emsp;&emsp;若每次上传图片都要打开 Backblaze 网站终归还是很麻烦，好在 PicGo 能够让整个过程自动化。PicGo 还提供了丰富的插件，可以实现自定义文件路径、文件名哈希化等功能。
<br>
&emsp;&emsp;设置 PicGo 作为 Backblaze 的图片上传工具，需要先打开 Backblaze Buckets 页面，在桶信息中记录下 Endpoint 的内容：
<br>
![test](https://img.locyoo.com/1194.png)
<br>
&emsp;&emsp;再在页面中找到 Application Keys 界面，选择 Add a New Application Key，填入 key 的名字：
<br>
![test](https://img.locyoo.com/1195.png)
<br>
在 Duration 一项可以设置 key 的有效期，过期后需要重新申请。选择提交后，页面就会给出生成的 keyID 和 applicationKey，将内容复制保存下来，一凡离开该页面就再也无法查看了。
<br>
![test](https://img.locyoo.com/1196.png)
<br>
安装好 PicGo 后，搜索并安装 s3 插件，打开 Amazon S3 的设置界面，填入先前保存下的信息，我的设置如下：
<br>
"aws-s3": {
    "accessKeyID": "Backblaze keyID",
    "secretAccessKey": "Backblaze applicationKey",
    "endpoint": "https://s3.us-west-000.backblazeb2.com",
    "bucketName": "a-complicated-name",
    "uploadPath": "{year}/{month}/{sha256}.{extName}",
    "urlPrefix": "https://cdn.leonis.cc/img/"
}
<br>
其中比较关键的是 accessKeyID、secretAccessKey、endpoint 三项，确保填写正确，另外不要忘了在 endpoint 前加上 https://。其余项则用于自定义图片路径和得到的 URL，具体配置可以参考插件仓库中的说明。