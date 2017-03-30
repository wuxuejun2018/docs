## 录播
> 需提供的配置信息：需要录制的拉流 URL    
> 录制方式：触发或定时录制    

录播的主要作用是将推流内容录制成文件，最终用于点播。
现支持录制成 MP4、FLV、TS、M3u8，默认录制成 mp4  格式文件，上传到又拍云存储，客户可以自定义上传到哪个存储服务。  

又拍录制系统会自动将录制下来的内容上传到又拍云存储后，可以根据云存储获取目录文件列表来获取相关录制文件列表。如果对录制文件格式有其他要求，可在又拍云处理中心对其进行相关处理，比如格式处理，视频拼接，详细请见[云处理文档](http://docs.upyun.com/cloud/)。

如果配置对 rtmp://play.com/live/stream 这条流进行录制，录制后文件具体路径为：  

```
upyun-live-recorder.b0.upaiyun.com/play.com/live/stream/recorder20160604163702.mp4  

其中 upyun-live-recorder 为存储空间， upyun-live-recorder.b0.upaiyun.com 为录制文件默认播放域名,  
play.com 为直播拉流域名，live 为接入点，stream 为流名，recorder 系统默认标识符名，  
20160604163702 为录制完成时间，mp4 为文件类型。录制完成后可以通过 post 方式回调给用户提供的回调地址。  

```
录制系统会将录制文件默认保存在该空间以这条流 URL 为路径的目录下，  
即 upyun-live-recorder.b0.upaiyun.com/play.com/live/stream/，使用又拍文件加速服务，直接可通过   http://client.com/play.com/live/stream/recorder20160604163702.mp4 来访问，client.com 为客户点播域名，需绑定在录制文件所有的云存储空间，该过程即对存储内容进行点播。

录制支持触发录制与定时录制两种方式，并且支持网络闪断重连后的文件合并，具体需要多长时间内的闪断进行合并，可配置，如要进行合并，断流前后的直播流分辨率需要保持一致。

> 需要录制的流协议支持：RTMP，HTTP-FLV。

### 触发录制
> 源站类型：又拍云源

指定配置某一条流为触发录制的流，则在这条流推流到又拍 CDN 的时候，又拍录制系统对其开始录制，当这推流断开后停止录制，再推流后又继续开始录制，以此循环。推流断开时，录制文件自动停止录制，下次再推流后，录制的文件名将与前一文件名不一样，这样同一条流会生成多个不同文件，并以文件名最后的时间表示其先后顺序。推流如果一直不断开，录制文件默认 2 小时切割一次。

> 触发录制需要在推流开始前配置才能生效，在推流过程中配置触发录制不生效。

### 定时录制
> 源站类型：全部  

指定配置某一条流的起止时间，比如从 2016-09-01 17：00：00 开始录制，并到 2016-09-01 22：00：00 结束，不管在这时间段内推流断开几次，录制系统将在开始录制的前 2 小时内，对其推流的所有流内容进行录制在同一文件名下，超过 2 小时部分会自动切割，录制成下一个文件。

### 回调
> 需提供接收回调的地址（建议为 URL）  

录制文件所在的路径以 post 请求返回给客户，具体的 json 格式为
```
{"timestamp": "2016-06-04 16:38:45",  
 "path": ["http://live-recorder.b0.upaiyun.com/play.com/live/stream/recorder20160604163702.mp4"]}
```
其中，timestamp 为发送 json 回调任务时间，path 为录制文件具体路径。

## 转码
> 源站类型：又拍云源  
> 提供要转码的原始流、转码匹配的后缀及转码模板。  

支持音视频流实时转码处理，通过转码模版可配置编码标准、分辨率、码率及输出流类型等流处理参数。
默认支持使用又拍云直播服务的 RTMP，HTTP-FLV 和 HLS 协议的流转码支持 12 种转码模板和客户自定义转码配置，详细模板信息,参考[视频转码预置模板](http://docs.upyun.com/cloud/attachment/ ) 支持自定义转码后缀，分隔符支持中划线（-）、下划线（_）和感叹号（!）。

支持触发式转码，需提前配置需要转码的流地址以及转码的触发后缀，如 需要转码的原始流为：http://play.com/live/stream 触发转码的后缀匹配为 -small，对应的转码模板为 540p（16:9） 当有用户请求 http://play.com/live/stream-small  时触发转码，当最后一个请求该转码流的用户断开连接后，停止转码。

```
示例：
原始流为 rtmp://play.com/live/stream 配置的匹配后续有 -small，对应转码模板 540p（16:9），  
则转码流请求分别为  
rtmp://play.com/live/stream-small  
http://play.com/live/stream-small.flv  
http://play.com/live/stream-small.m3u8    
```

## 禁播
> 源站类型：又拍云源  

互动直播业务特性为：直播视频内容由主播实时推送至服务器，若主播传播非法内容，则影响巨大。因此厂商需要对直播视频内容进行实时监控，当出现非法内容时，及时禁止非法内容的传播。  
秒级禁播则是针对该业务场景而研发的功能，由又拍提供页面配置和调用 API 禁播两种方式。当客户发现某直播视频内容非法时，实时断开主播推流，从而阻止非法内容的传播。  
禁播方式支持立即禁播，解除禁播，以及禁播一段时间后自动解除。  

## 截图
> 源站类型：又拍云源

直播截图服务支持将直播视频以设定的时间间隔和尺寸大小进行截图，并以 .jpg 格式保存在存储空间。如果对截图文件有处理需求，可在又拍云处理中心对其进行相关处理操作，详情请见[云处理文档](http://docs.upyun.com/cloud/)。
### 单次截图
仅在直播推流时进行单次截图，截取推流的第一帧。同一条流，重推后将重新截取首帧，并覆盖上次推流的截图。该截图模式适用于静态封面。  
如果对服务号为 upyun-live 的直播空间开启单次截图，选择存储空间为 upyun-live-pic ，截图后文件的具体路径为
```
upyun-live-pic.b0.upaiyun.com/upyun-live/live/stream.jpg
其中 upyun-live-pic 为存储空间， upyun-live-pic.b0.upaiyun.com 为截图文件默认访问域名， 
upyun-live 为直播空间，live 为接入点， stream 为流名。 
```
### 多次截图
开启多次截图，可设置固定时间间隔，在推流过程中，以一定时间间隔进行截图，直到推流结束，适用于动态封面和内容审核。  

如果对服务号为 upyun-live 的直播空间开启多次截图，选择存储空间为 upyun-live-pic ，截图保存路径及命名可设置为 {直播加速服务名}/{接入点}/{流名}{年}{月}{日}{时}{分}{秒}.jpg，则截图后文件的具体路径为
```
upyun-live-pic.b0.upaiyun.com/upyun-live/live/stream20170101121212.jpg  
其中 upyun-live-pic 为存储空间， upyun-live-pic.b0.upaiyun.com 为截图文件默认访问域名，
upyun-live 为直播空间，live 为接入点， stream 为流名，
201701011212 为 2017 年 1 月 1 日 12 点 12 分 12 秒。
```
> 截图保存路径及命名详情见后台配置页面。  

### 回调
> 需提供接受回调的地址（建议为 URL）

直播截图文件所在路径以 post 请求返回给客户，具体的 json 格式为
```
{
  "bucket": "upyun-live", // 直播服务名 
  "app": "live", // 直播接入点名称 
  "stream": "stream", // 直播流名 
  "vhost": "push.com", // 推流域名 
  "path": "http://upyun-live-pic.b0.upaiyun.com/upyun-live/live/stream.jpg", // 截图存储路径 
  "timestamp": "2017-01-01 12:12:12", // 截图时间 
  "status": 200, // 截图状态 
  "error": ""  // 如果截图成功，即 status 为 200 ，error 字段为空字符串，否则 error 字段会给出错误信息   
}
```

