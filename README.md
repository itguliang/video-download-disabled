# video-download-disabled

实现视频可播放不能下载，禁止右键下载、F12源码打开链接下载

[查看源码](https://github.com/itguliang/video-download-disabled)

在线demo：

[缓存完再播放](https://itguliang.github.io/video-download-disabled/playAfterBuffered.html)

[边播放边缓存](https://itguliang.github.io/video-download-disabled/playWhenBuffering.html)

### Video 禁止鼠标右键下载
```html
<!-- 添加 oncontextmenu="return false" -->
<video id="videoDemo" controls preload="auto" oncontextmenu="return false"></video>
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