# video-download-disabled

### 效果

- 实现视频可播放不能下载，禁止右键下载、F12源码打开链接下载
- 实现只在当前窗口播放，切换窗口、窗口最小化、窗口被遮挡停止播放，恢复后继续播放

[查看源码](https://github.com/itguliang/video-download-disabled)

在线demo：

[缓存完再播放](https://itguliang.github.io/video-download-disabled/playAfterBuffered.html)

[边播放边缓存](https://itguliang.github.io/video-download-disabled/playWhenBuffering.html)

### Video 禁止鼠标右键下载
```html
<!-- 添加 oncontextmenu="return false" -->
<video src="地址" controls preload="auto" oncontextmenu="return false"></video>
```

### 禁止源码打开链接下载

主要使用 [MediaSource](https://developer.mozilla.org/zh-CN/docs/Web/API/MediaSource) 和 [createObjectURL](https://developer.mozilla.org/en-US/docs/Web/API/URL/createObjectURL) 实现

html:
```html
<video id="videoDemo" controls preload="auto" oncontextmenu="return false" ></video>
```
js:
```javascript
//video/webm;codecs = "vp8,vorbis"   表示webm视频容器中的vp8视频编解码器和vorbis音频编解码器3
//video/ogg;codecs = "theora,vorbis" 表示ogg视频容器中的theora视频编解码器和vorbis音频编解码器
//video/mp4;codecs = "avc1.42E01E,mp4a.40.2" 表示基本的MEPG-4视频容器中的H.264视频编解码器和ACC音频编解码器
//video/mp4;codecs = "avc1.64001E,mp4a.40.2" 表示高质量的MEPG-4视频容器中的H.264视频编解码器和ACC音频编解码器 
var video = document.getElementById("videoDemo");

    //mp4 格式 跟文件也有关
    var assetURL = "demo.mp4";
    var mimeCodec = 'video/mp4; codecs="avc1.42E01E, mp4a.40.2"';
    
    //webm 格式
    // var assetURL = "demo.webm";
    // var mimeCodec = 'video/webm;codecs="vorbis,vp8"';
    
    if ("MediaSource" in window && MediaSource.isTypeSupported(mimeCodec)) {
      var mediaSource = new MediaSource();
      video.src = URL.createObjectURL(mediaSource);
      mediaSource.addEventListener("sourceopen", sourceOpen);
    } else {
      console.error("Unsupported MIME type or codec: ", mimeCodec);
    }

    function sourceOpen() {
      console.log(this); // open
      var mediaSource = this;
      var sourceBuffer = mediaSource.addSourceBuffer(mimeCodec);
      fetchAB(assetURL, function(buf) {
        console.log(buf);
        console.log(sourceBuffer);
        sourceBuffer.addEventListener("updateend", function() {
          console.log(mediaSource);
          mediaSource.endOfStream();
          //video.play();  //这里会报错就去掉了
          console.log(mediaSource.readyState); // ended
        });
        sourceBuffer.appendBuffer(buf);
      });
    }
    function fetchAB(url, cb) {
      console.log("fetchAB----",url);
      var xhr = new XMLHttpRequest();
      xhr.open("get", url);
      xhr.responseType = "arraybuffer";
      xhr.onload = function() {
        console.log(xhr.response);
        cb(xhr.response);
      };
      xhr.send();
    }
```

待解决: 发现network还是可以看到视频链接的并且右键还是可以打开并且下载

###  `document.visibilityState` 监听浏览器最小化

`document.hidden`：表示页面是否隐藏的布尔值。页面隐藏包括 页面在后台标签页中 或者 浏览器最小化 （注意，页面被其他软件遮盖并不算隐藏，比如打开的 sublime 遮住了浏览器）。

`document.visibilityState`：表示下面 4 个可能状态的值
`hidden`：页面在后台标签页中或者浏览器最小化
`visible`：页面在前台标签页中
`prerender`：页面在屏幕外执行预渲染处理 document.hidden 的值为 true
`unloaded`：页面正在从内存中卸载

`Visibilitychange`事件：当文档从可见变为不可见或者从不可见变为可见时，会触发该事件。

这样，我们可以监听 `Visibilitychange` 事件，当该事件触发时，获取 `document.hidden` 的值，根据该值进行页面一些事件的处理。

```javascript
var videoPausedStatus;
document.addEventListener('visibilitychange', function () {
  var isHidden = document.hidden;
  if (isHidden) {
    videoPausedStatus = true;
    if (!video.paused) {
      videoPausedStatus = false;
      video.pause();
    }
  } else {
    if (!videoPausedStatus) {
      video.play();
    }
  }
});
// //IE
// document.addEventListener('msvisibilitychange',function(){
//   console.log(document.msVisibilityState);
// });
// //FF
// document.addEventListener('mozvisibilitychange',function(){
//   console.log(document.mozVisibilityState);
// });
// //chrome
// document.addEventListener('webkitvisibilitychange',function(){
//   console.log(document.webkitVisibilityState);
// });
```

### `document.hasFocus()` 判断当前页面是否被激活
解决不能监听页面被其他软件遮盖
```javascript
setInterval(function () {
  if (document.hasFocus() != pageFocus) {
    pageFocus = document.hasFocus();
    if (!pageFocus) {
      if (!video.paused) {
        videoPausedStatus = false;
        video.pause();
      }
    } else {
      if (!videoPausedStatus) {
        video.play();
      }
    }
  }
}, 1000);
```