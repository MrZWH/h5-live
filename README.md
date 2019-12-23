# h5-live-video

h5 直播

## 大前端

对于大前端来说，需要掌握以下至少一种技能：

- 服务类
  - Node.js、express.js、koa.js
- 3d 数据图像
  - three.js
- 二维图像
  - d3.js、raphael.js、echart.js
- 视频
  - video.js、hls.js、flv.js

## 怎么学

- 自己摸索不如前人指路
- 技术方案分分钟复制，不再烦恼
- 在巨人的肩膀上渐行渐远

## 直播原理

流程图：
![流程图](https://github.com/MrZWH/MrZWH.github.io/blob/master/images/%E7%9B%B4%E6%92%AD%E6%B5%81%E7%A8%8B%E5%9B%BE.PNG?raw=true)

收集的数据一开始是以原始二进制流的形式，通过 websocket 也好 http 也好，上传到服务器。原始的流不能用播放器播放，必须采用一定的协议做编码。H.264(视频的编码)、AAC（音频的编码）。
视频格式与浏览器兼容：
![流程图](https://github.com/MrZWH/MrZWH.github.io/blob/master/images/%E8%A7%86%E9%A2%91%E6%A0%BC%E5%BC%8F%E5%92%8C%E6%B5%8F%E8%A7%88%E5%99%A8%E6%94%AF%E6%8C%81.PNG?raw=true)
YouTube 使用的 webm，流式的视频格式，hls 标准来说不是一种视频格式，是一种apple公司推出的视频协议，对视频来说是 ts 文件`.ts`格式，flv b站的视频格式，是早期的 flash 的视频格式，b站的 h5 播放器也能播放 flv 格式的，为什么呢？

## 直播协议

整个行业里做直播最常用的三种协议：

- HLS 协议
  - 视频格式是 ts
- RTMP 协议
  - 视频格式是 flv
- HTTP-FLV 协议
  - 视频格式是 flv

点播：多媒体播放器的术语，对播放器来说分点播和直播。

### HLS 协议

![image](https://github.com/MrZWH/MrZWH.github.io/blob/master/images/hls%E5%8D%8F%E8%AE%AE.PNG?raw=true)
对于 hls 协议，它会给一个 M3U8 文件。直播跟点播最大的区别：不会一下子给视频地址，会给一个M3U8 索引文件，浏览器会根据 索引文件的片段时长自动跟新 m3u8 文件。

#### HLS 协议快速预览

1. 安装 Safari 浏览器
2. 用 Safari 浏览器打开下面任意地址均可

- <http://live.streamingfast.net/osmflivech4.m3u8>
- <http://live.streamingfast.net/osmflivech5.m3u8>
- <http://live.streamingfast.net/osmflivech1.m3u8>

#### m3u8 文件

![image](https://user-images.githubusercontent.com/24401665/71319786-f1200000-24dd-11ea-8ce9-78e98d5f4e7e.png)

m3u8 文件里面存放的是索引，每个索引对应这一个 ts 文件，但有时 m3u8 文件里可以嵌套 m3u8 文件。有时候拿到一个 m3u8 文件播放不了需要排查浏览器是否支持。

m3u8 文件还细分为几类：

- live playlist 动态列表，多用于直播
- event playlist 静态列表，在直播行业里几乎是见不到的，标准中有提到，实际应用中没人用
- vod playlist 全量列表 （vod 英文意思“点播”）

m3u8 文件动态列表里面的内容：
![m3u8 content](https://github.com/MrZWH/MrZWH.github.io/blob/master/images/m3u8%E6%96%87%E4%BB%B6%E5%86%85%E5%AE%B9.PNG?raw=true)

m3u8 文件是纯文本文件，不是多媒体的流。  
文件内容第一行是 m3u8 文件的版本，使用时需要看浏览器是否支持该版本。

m3u8 文件静态列表里面的内容：

```text
#EXTM3U
#EXT-X-VERSION:6
#EXT-X-TARGETDURATION:10
#EXT-X-MEDIA-SEQUENCE:0
#EXT-X-PLAYLIST-TYPE:EVENT
#EXTINF:9.9001,
http://media.example.com/wifi/segment0.ts
#EXTINF:9.9001,
http://media.example.com/wifi/segment1.ts
#EXTINF:9.9001,
http://media.example.com/wifi/segment2.ts
```

m3u8 文件全量列表里面的内容：

```text
#EXTM3U
#EXT-X-VERSION:6
#EXT-X-TARGETDURATION:10
#EXT-X-MEDIA-SEQUENCE:0
#EXT-X-PLAYLIST-TYPE:VOD
#EXTINF:9.9001,
http://media.example.com/wifi/segment0.ts
#EXTINF:9.9001,
http://media.example.com/wifi/segment1.ts
#EXTINF:9.9001,
http://media.example.com/wifi/segment2.ts
#EXT-X-ENDLIST
```

HTL 是直播里应用最广泛的协议  

#### ts 文件

![ts file](https://github.com/MrZWH/MrZWH.github.io/blob/master/images/ts%E6%96%87%E4%BB%B6.PNG?raw=true)

第一个 ts 文件会有一个 PAT 包，这个包告诉你要去找一个 PMT 的包，PMT 包告诉你后面会解析到 ts 包，告诉你这些 ts 包哪些是视频哪些是音频，很多个ts 组成一个叫 PES 的东西。  
浏览器解析一个视频需要知道视频帧和音频帧，哪些ts 包能组成一个帧？需要解析 ts 规范，ts 中有个 header 它会告诉你这些信息。

#### RTMP 协议

RTMP 是 Real TIme Messaging Protocol（实时消息传输协议）的首字母缩写。该协议基于 TCP ，是一个协议族，包括 RTMP 基本协议以及 RTMPT/RTMPS/RTMPE 等多种变种。RTMP 是一种设计用来进行实时数据通信的网络协议，主要用来在 Flash、AIR 平台和支持 RTMP 协议的流媒体/交互服务器之间进行音视频和数据通信。

视频音频的采集以PC 端为主，如果不是通过 web 的方式来做而是通过客户端的方式来做，这时基本走 RTMP 协议，如果采集端用 web 端来做，比如 h5 ，那么它的协议叫做 webRTC，这是两种不同的技术方案。通常给主播的是客户端采用RTMP 效率比较高。

![rtmp通信过程](https://github.com/MrZWH/MrZWH.github.io/blob/master/images/rtmp%E9%80%9A%E4%BF%A1%E8%BF%87%E7%A8%8B.PNG?raw=true)
RTMP 协议在传输过程中也是 flv 视频格式的，只是通信的手段不一样，tcp 通信有三次应答，hls 是 http 的不需要走 3次应答的过程。所以使用 rtmp 时需要注意跟用户端的应答处理。

#### HTTP-FLV 协议

![HTTP-FLV通信过程](https://github.com/MrZWH/MrZWH.github.io/blob/master/images/http-flv%E9%80%9A%E4%BF%A1%E8%BF%87%E7%A8%8B.PNG?raw=true)

RTMP 的升级版，RTMP 比 hls 用起来更复杂，hls 实时性差，因为请求一个 m3u8 文件会请求到多个 ts 文件，所以延时的程度跟 ts 文件的切片分片数量有关系。HTTP-FLV 结合了 hls http 请求的优点和 rtmp 低延时的特性，中间建立了一个 flv 长连接。因为中间建立了一个长连接所以通信过程没那么复杂，唯一跟 rtmp 的区别就是播放器跟 cdn 建立的连接是直接的 http 请求。

相对RTMP的优点：

- 可以在一定程度上避免防火墙的干扰（例如，有的机房只允许 80 端口通过）。
- 可以很好的兼容 HTTP 302 的跳转，做到灵活调度。请求 cdn 时如果cdn 的某一个节点出现问题的时候必须要 302 重定向到其它的节点，如果这个都做不到的话就不够灵活会请求失败。
- 可以使用 HTTPS 做加密通道
- 可以很好的支持移动端（Android，IOS）

### 直播原理总结

- 了解直播协议的分类
  - 三种
- 理解对应协议的通信原理
- 理解直播协议之间的区别（hls 好用但是延时大，RTMP 实时性比较好但是使用起来比较复杂，HTTP-FLV结合两者的优点但是视频格式是flv的）
- 掌握不同场景采用什么协议
  - 采集这一块用客户端来做的，通常是使用 rtmp 协议，因为后端对 tcp 通信比较了解。如果对延时不那么敏感，像斗鱼直播都是采用的 hls 协议，传输的是 ts 文件而不是 flv，它也不需要建立一个长连接，考虑到服务器的并发都是有好处的。
  
## video 详解

在 video src 中有些地址回事 `blob:https://...`开头的地址，这是虚拟地址，下载不到的。
![video](https://github.com/MrZWH/MrZWH.github.io/blob/master/images/video%E5%B1%9E%E6%80%A7%E5%92%8C%E6%96%B9%E6%B3%95.PNG?raw=true)
![video](https://github.com/MrZWH/MrZWH.github.io/blob/master/images/video%E6%A0%87%E7%AD%BE%E7%9A%84%E4%BA%8B%E4%BB%B6.PNG?raw=true)

### 实战 h5live-demo

去 <https://v.qq.com>下载老的格式为 mp4 的视频，右键视频地址新窗口打开，下载。

Web Server Chrome（chrome 应用商店插件） 选中h5live-demo 文件夹，就能启动一个服务。

在 video 标签加上属性`controlslist="nodownload"`就会禁用下载，还可以加 nofullscreen 去掉全屏。

移动端非静音状态下的视频不允许自动播放。
没有 source 标签时，可以`video.src`获取视频地址，当有 source 时通过`video.currentSrc`获取视频地址。上报失效视频地址可以通过遍历 source dom 比对 currentSrc 将错误地址上报。

#### 事件

当video创建之后拿到的 duration 是 NaN，然后会出发 durationchange 事件，加载完元信息之后才会拿到视频的 duration，这个跟视频的格式有关系。有时候视频第一次变化的时候 duration只是视频的一部分，durationchange会触发多次。

视频的元数据  
先出发 durationchange 再触发 loadedmetadata

在监听到 canplaythrough、canplay 时去播放

## 直播源的制作

方法一 Nginx + ffmpeg：

- 安装 Nginx
  - Mac: brew install nginx-full --with-rtmp-module
  - Windows: <http://nginx.org/en/docs/windows.html>
  - 进入网址点 download

cd c:\
unzip nginx-1.17.6.zip
cd nginx-1.17.6
start nginx

- 安装 ffmpeg
  - Mac: brew install ffmpeg
  - Windows: <https://ffmpeg.zeranoe.com/builds/>
验证

输入：ffmpeg

- 配置 Nginx
- 准备视频
- 利用 ffmpeg 推流

rtmp 推流：

```shell
ffmpeg -re -i test.mp4 -vcodec libx264 -acodec aac -f flv rtmp://localhost:1935/rtmplive/rtmp
```

hls 推流：

```shell
ffmpeg -re -i test.mp4 -vcodec libx264 -acodec aac -f flv rtmp://localhost:1935/hls/stream
```

然后用 safari 浏览器访问<localhost:8080/hls/stream.m3u8>

安装 VLC 和 Safari 浏览器

- mac 系统：<https://www.videolan.org/vlc/index.zh.html>
- Windows: <https://www.videolan.org/vlc.index.zh.html>

方法二 集成服务：

1. 下载服务
2. 安装服务
3. 准备源视频
4. 开启服务 open server
5. 利用 ffmpeg 推流

- ffmpeg -re -i test.mp4 -c copy -f flv rtmp://localhost:1935/live/movie

验证：

- RTMP: rtmp://localhost:1935/live
- FLV: <http://127.0.0.1:7001/live/movie.flv>
- HLS: <http://127.0.0.1:7002/live/movie.m3u8>

## H5 直播演练

- 播放器选型
  - VIDEOJS(<https://github.com/videojs/video.js>) 国外比较好的视频框架，有自定义UI，有很多插件，播hls、弹幕、清晰度切换、快捷键 插件地址<https://videojs.com/plugins/>
  - hls.js(<https://github.com/video-dev/hls.js>) 非常小巧的适用于使用 hls 协议的直播和点播
  - flv.js(<https://github.com/bilibili/flv.js>) b 站开源的 flv 格式的播放器，http-flv 协议的直播
  - rtmp协议的在实际的互联网直播中不常用
- 实例开发演示
  
### elint 配置

npm 安装了 eslint 后可以输入以下命令初始化选择已有的 eslint 配置：

```shell
eslint --init
```

## 微信小程序直播

- 项目搭建
- 直播组件配置
  - live-player
  - 目前是支持 flv、rtmp 格式
- 真机调试

## 课程总结

- 直播的工作原理
  - 协议
  - 通信
  - 区别
- video 的深入理解
  - 属性
  - 方法
  - 事件
- 直播流的制作
  - 高级
  - 简易
  - 现在的云服务都提供了直播流的推送服务
- h5 直播实战
  - 选型
  - 方法
  - 进阶
- 微信小程序直播实战
  - 配置
  - 推流
  - 调试

监控？弹幕？刷礼物？点播？省流量？自定义皮肤？

<https://github.com/cucygh/h5live><https://github.com/cucygh>