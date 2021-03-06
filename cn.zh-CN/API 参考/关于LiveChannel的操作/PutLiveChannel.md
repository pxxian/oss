# PutLiveChannel {#reference_d34_zcd_xdb .reference}

通过RTMP协议上传音视频数据前，必须先调用该接口来创建一个LiveChannel。该接口会返回RTMP推流地址，以及对应的播放地址。

用户可以使用返回的地址进行推流、播放。另外，用户可以根据该LiveChannel的名称来发起相关的操作，如查询推流状态、查询推流记录、禁止推流等。

## 请求语法 {#section_lmy_c2d_xdb .section}

```
PUT /ChannelName?live HTTP/1.1
Host: BucketName.oss-cn-hangzhou.aliyuncs.com
Date: GMT date
Content-Length: Size
Authorization: SignatureValue
<?xml version="1.0" encoding="UTF-8"?>
<LiveChannelConfiguration>
  <Description>ChannelDescription</Description>
  <Status>ChannelStatus</Status>
  <Target>
     <Type>HLS</Type>
     <FragDuration>FragDuration</FragDuration>
     <FragCount>FragCount</FragCount>
     <PlayListName>PlaylistName</PlayListName>
  </Target>
  <Snapshot>
    <RoleName>Snapshot ram role</RoleName>
    <DestBucket>Snapshot dest bucket</DestBucket>
    <NotifyTopic>Notify topic of MNS</NotifyTopic>
    <Interval>Snapshot interval in second</Interval>
  </Snapshot>
</LiveChannelConfiguration>
```

## 请求元素\(Request Elements\) {#section_f54_f2d_xdb .section}

|名称|类型|描述|是否必需|
|:-|:-|:-|:---|
|LiveChannelConfiguration|容器|保存LiveChannel配置的容器。子节点：Description、Status、Target

父节点：无

|是|
|Description|字符串|LiveChannel的描述信息，最长128字节。子节点：无

父节点：LiveChannelConfiguration

|否|
|Status|枚举字符串|指定LiveChannel的状态。子节点：无

父节点：LiveChannelConfiguration

有效值：enabled、disabled

默认值：enabled

|否|
|Target|容器|保存转储配置的容器。子节点：Type、FragDuration、FragCount、PlaylistName

父节点：LiveChannelConfiguration

|是|
|Type|枚举字符串|指定转储的类型。子节点：无

父节点：Target

有效值：HLS

|是|
|FragDuration|字符串|当Type为HLS时，指定每个ts文件的时长（单位：秒），取值范围为\[1, 100\]的整数。子节点：无

父节点: Target

默认值：5

|否|
|FragCount|字符串|当Type为HLS时，指定m3u8文件中包含ts文件的个数，取值范围为\[1, 100\]的整数。子节点：无

父节点：Target

默认值: 3

|否|
|PlaylistName|字符串|当Type为HLS时，指定生成的m3u8文件的名称，必须以”.m3u8”结尾，长度范围为\[6, 128\]。子节点：无

父节点：Target

默认值：playlist.m3u8

|否|
|Snapshot|容器|保存高频截图操作Snapshot 选项的容器。子节点：RoleName，DestBucket，NotifyTopic，Interval，PornRec

父节点：Snapshot

|否|
|RoleName|字符串|用于高频截图操作的角色名称，要求有DestBucket的写权限和向NotifyTopic发消息的权限。子节点：无

父节点：Snapshot

|否|
|DestBucket|字符串|保存高频截图目标Bucket，要求与当前Bucket是同一个Owner。子节点：无

父节点：Snapshot

 |否|
|NotifyTopic|字符串|用于通知用户高频截图操作结果的MNS的Topic。子节点：无

父节点：Snapshot

|否|
|Interval|数字|高频截图的间隔长度，单位为s，如果该段间隔时间内没有关键帧（I帧），那么该间隔时间不截图。子节点：无

父节点：Snapshot

|否|

## 细节分析 {#section_gfp_5fd_xdb .section}

-   ChannelName必须符合ObjectName的命名规范，另外，ChannelName不能包含”/“。
-   FragDuration和FragCount的默认值只有在两者都未指定时才会生效；指定了其中一个，则另一个的值也必须指定。
-   转储类型为HLS时，OSS会在生成每个ts文件后，更新m3u8文件；m3u8文件中最多包含最近的FragCount个ts文件。
-   转储类型为HLS时，写入当前ts文件的音视频数据时长达到FragDuration指定的时长后，OSS会在收到下一个关键帧的时候切换到下一个ts文件；如果max\(2\*FragDuration, 60s\)后仍未收到下一个关键帧，OSS强制切换文件，此时可能引起播放时卡顿。

## 响应元素 {#section_znt_xfd_xdb .section}

|名称|类型|描述|
|:-|:-|:-|
|CreateLiveChannelResult|容器|保存CreateLiveChannel请求结果的容器。子节点：PublishUrls,PlayUrls

父节点：无

|
|PublishUrls|容器|保存推流地址的容器。子节点：Url

父节点：CreateLiveChannelResult

|
|Url|字符串|推流地址。子节点：无

父节点：PublishUrls

|
|PlayUrls|容器|保存推流地址的容器。子节点：Url

父节点：CreateLiveChannelResult

|
|Url|字符串|播放地址。子节点：无

父节点：PlayUrls

|

## 细节分析 {#section_fjs_jgd_xdb .section}

-   推流地址是未加签名的url，如果bucket acl非public-read-write，那么需要首先进行签名才能进行访问。
-   播放地址是未加签名的url，如果bucket acl为private，那么需要首先进行签名才能进行访问。

## 实例 {#section_qzp_ngd_xdb .section}

**请求示例**

```
PUT /test-channel?live HTTP/1.1
Date: Wed, 24 Aug 2016 11:11:28 GMT
Content-Length: 333
Host: test-bucket.oss-cn-hangzhou.aliyuncs.com
Authorization: OSS YJjHKOKWDWINLKXv:hvwOZJRh8toAj3DZvtsuPgf+agA=
<?xml version="1.0" encoding="utf-8"?>
<LiveChannelConfiguration>
    <Description/>
    <Status>enabled</Status>
    <Target>
        <Type>HLS</Type>
        <FragDuration>2</FragDuration>
        <FragCount>3</FragCount>
    </Target>
    <Snapshot>
        <RoleName>role_for_snapshot</RoleName>
        <DestBucket>snapshotdest</DestBucket>
        <NotifyTopic>snapshotnotify</NotifyTopic>
        <Interval>1</Interval>
     </Snapshot>
</LiveChannelConfiguration>
```

**返回示例**

```
HTTP/1.1 200
content-length: 259
server: AliyunOSS
x-oss-server-time: 4
connection: close
x-oss-request-id: 57BD8419B92475920B0002F1
date: Wed, 24 Aug 2016 11:11:28 GMT
x-oss-bucket-storage-type: standard
content-type: application/xml
<?xml version="1.0" encoding="UTF-8"?>
<CreateLiveChannelResult>
  <PublishUrls>
    <Url>rtmp://test-bucket.oss-cn-hangzhou.aliyuncs.com/live/test-channel</Url>
  </PublishUrls>
  <PlayUrls>
    <Url>http://test-bucket.oss-cn-hangzhou.aliyuncs.com/test-channel/playlist.m3u8</Url>
  </PlayUrls>
</CreateLiveChannelResult>
```

