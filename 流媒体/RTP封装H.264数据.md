# RTP封装H.264数据



## H.264数据结构特点

H264 原始码流是由一个接一个 NALU（NAL Unit） 组成，它的功能分为两层，VCL（Video Coding Layer）视频编码层和 NAL（Network Abstraction Layer）网络提取层。

> VCL：包括核心压缩引擎和块、宏块和片的语法级别定义，设计目标是尽可能地独立于网络进行高效的编码；
>
> NAL：负责将 VCL 产生的比特字符串适配到各种各样的网络和多元环境中，覆盖了所有片级以上的语法级别；

NAL是 H.264 为适应网络传输应用而制定的一层数据打包操作。传统的视频编码算法编完的视频码流在任何应用领域下（无论用于存储、传输等）都是统一的码流模式，视频码流仅有视频编码层 VCL（Video Coding Layer）。而 H.264 可根据不同应用增加不同的 NAL 片头，以适应不同的网络应用环境，减少码流的传输差错。

在 VCL 进行数据传输或存储之前，这些编码的 VCL 数据，被映射或封装进NAL单元（NALU）。

### NALU Header & RBSP 结构

```txt
一个 NALU = 一组对应于视频编码的 NALU 头部信息 + 一个原始字节序列负荷（RBSP，Raw Byte Sequence Payload）

    +------------+----------------+------------+---
 ...| NAL header |      RBSP      | NAL header | RBSP  ...
    +------------+----------------+------------+---
```

一个原始的 H.264 NALU 单元常由 [StartCode] [NALU Header] [NALU Payload] 三部分组成，其中 Start Code 用于标示这是一个NALU 单元的开始，必须是 `00 00 00 01`。**实际原始视频图像数据保存在 VCL 分层的 NAL Units 中**。

对于一个NALU的数据如下

```txt
  0                   1                   2                   3
  0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |F|NRI|   type  |         RBSP(Single NALU unit)                |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 
 对于header中的type有如下几种
 1-23 NAL单元 单个 NAL 单元包
 24  STAP-A  单一时间的组合包
 25  STAP-B  单一时间的组合包
 26  MTAP16  多个时间的组合包
 27  MTAP24  多个时间的组合包
 28  FU-A   分片的单元
 29  FU-B   分片的单元
 30-31 没有定义
```



## RTP协议

RTP用来为IP网上的音频、视频、传真等多种需要实时传输的多媒体数据提供端到端的实时传输服务，RTP协议详细说明了在互联网上传输音频和视频的数据包格式。RTP为Internet上端到端的实时传输提供时间信息和流同步，但不能证服务质量，服务质量由RTCP来提供。因此，用于流媒体系统时，RTP和RTCP一起使用，而且它是创建在UDP协议上的。

### 报文格式

RTP报文由两部分组成：报头和有效载荷。RTP报头格式其中：

* V：RTP协议的版本号，占2位，当前协议版本号为2。
* P：填充标志，占1位，如果P=1，则在该报文的尾部填充一个或多个额外的八位组，它们不是有效载荷的一部分。
* X：扩展标志，占1位，如果X=1，则在RTP报头后跟有一个扩展报头。
* CC：CSRC计数器，占4位，指示CSRC 标识符的个数。 【1Byte】
* M：标记，占1位，不同的有效载荷有不同的含义，它用来允许在比特流中标记重要的事件,如对于视频，标记一帧的结束；对于音频，标记会话的开始。
* PT：有效载荷类型，占7位，用于说明RTP报文中有效载荷的类型，如GSM音频、JPEM图像等。
* 序列号(sequence number)：16 比特，每发送一个 RTP 数据包,序列号加 1，接收端可以据此检测丢包和重建包序列。序列号的初始值是随机的(不可预测),以使即便在源本身不加密时(有时包要通过翻译器,它会这样做)，对加密算法泛知的普通文本攻击也会更加困难。
* 时戳(Timestamp)：占32位，时戳反映了该RTP报文的第一个八位组的采样时刻。接收者使用时戳来计算延迟和延迟抖动，并进行同步控制。
* 同步信源(SSRC)标识符：占32位，用于标识同步信源。该标识符是随机选择的，参加同一视频会议的两个同步信源不能有相同的SSRC。
* 特约信源(CSRC)标识符：每个CSRC标识符占32位，可以有0～15个。每个CSRC标识了包含在该RTP报文有效载荷中的所有特约信源。



这里的同步信源是指产生媒体流的信源，它通过RTP报头中的一个32位数字SSRC标识符来标识，而不依赖于网络地址，接收者将根据SSRC标识符来区分不同的信源，进行RTP报文的分组。特约信源是指当混合器接收到一个或多个同步信源的RTP报文后，经过混合处理产生一个新的组合RTP报文，并把混合器作为组合RTP报文的SSRC，而将原来所有的SSRC都作为CSRC传送给接收者，使接收者知道组成组合报文的各个SSRC。

```txt
  0                   1                   2                   3
  0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |V=2|P|X|  CC   |M|     PT      |       sequence number         |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |                           timestamp                           |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |           synchronization source (SSRC) identifier            |
 +=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+
 |            contributing source (CSRC) identifiers             |
 :                             ....                              :
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

```



## RTP封装NALU

由于UDP数据报长度超过1500字节时(俗称MTU)，会自动拆分发送，增大了丢包概率，那么去除UDP数据报头以及RTP的Header部分，一般设置Payload部分最大长度为1400字节即可，那么对H264的NALU单元打RTP就意味着3种情况。



* 第一种：RTP包里只包含一个NALU,(它的数据小于1400字节)。RTP包里只包含一个NALU的情况，这时的RTP负载（Payload）部分如下图

```txt
  0                   1                   2                   3
  0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |F|NRI|   type  |                                               |
 +-+-+-+-+-+-+-+-+                                               +
 |                  RBSP(Single NALU unit)                       |
 +                         		   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |                                 :...OPTIONAL RTP padding      |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```



* 第二种：RTP包里包含N个NALU,(N个NALU的数据累加小于1400字节)



* 第三种：NALU数据大于1400字节, (比如5400字节，5400/1400>3.8,要拆分分4个RTP包)

```
  0                   1                   2                   3
  0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |  FU indicator |   FU header   |                               |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+                               +
 |                          FU payload                           |
 +                                 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |                                 :...OPTIONAL RTP padding      |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  
  0 1 2 3 4 5 6 7      	 0 1 2 3 4 5 6 7 
 +-+-+-+-+-+-+-+-+    	+-+-+-+-+-+-+-+-+
 |F|NRI|   type  |    	|S|E|R|  type   |
 +-+-+-+-+-+-+-+-+    	+-+-+-+-+-+-+-+-+
   FU indicator            FU header
```

但是我们处理H264数据时，一般是对NALU逐一进行处理的,因此我们只考虑第一和第三种情况。

 

对于第一种，从内存分布上可以理解为 RTP PlayLoad = [PlayLoad_head] + [RBSP]，我们知道一个 NALU = [NAL_head] + [RBSP]。

> PlayLoad_head 和 NAL_head 都是一个字节，结构相同。在这种情况下，PlayLoad_head 和 NAL_head的结构和值都是一样的，因此RTP PlayLoad = NALU= [NAL_head] + [RBSP]




我们发现FU Indicator的结构和NALU头结构是一样的，不过值不完全一样

> FU Indicator->F    = NALU头结构-> F
> FU Indicator->NRI  = NALU头结构->NRI
> FU Indicator->Type  = 28 (也就是扩展类型,FU-A)



另外

> FU header->Type = NALU头结构->Tpye
> FU header->S = 开始位,设置成1,指示分片NAL单元的开始,也就是开始包
> FU header->E = 结束位,设置成1,指示分片NAL单元的结束,也就是尾包
> FU header->R = 保留位必须设置为0











