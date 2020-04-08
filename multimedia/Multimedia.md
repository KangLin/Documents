多媒体
------
作者：康林(kl222@126.com)


### web 播放器
#### video.js

- videojs是基于html5的，所以只对mp4，webm，ogv三种格式支持。
- HLS: videojs是7.0版本以上的，集成VHS协议库(http-streaming)，可播放HLS流媒体视频。
- rtmp: 如果要实现rtmp流媒体视频需引入videojs-flash的支持。

- video.js：https://github.com/videojs/video.js
- http-streaming: https://github.com/videojs/http-streaming
- videojs-flash: https://github.com/videojs/videojs-flash

```javascript
<video-js id=vid1 width=600 height=300 class="vjs-default-skin" controls>
  <source
     src="https://example.com/index.m3u8"
     type="application/x-mpegURL">
</video-js>
<script src="video.js"></script>
<script src="videojs-http-streaming.min.js"></script>
<script src="videojs-flash.min.js"></script>
<script>
var player = videojs('vid1');
player.play();
</script>
```javascript

#### mediaelement

MediaElementPlayer：HTML5 video和 audio播放器,MediaElement.js支持IE11 +，MS Edge，Chrome，Firefox，Safari，iOS 8+和Android 4.0+。
 
- 地址：https://github.com/mediaelement/mediaelement
