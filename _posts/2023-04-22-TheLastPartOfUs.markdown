---
layout: post
title:  "The Last Part Of Us(Part I)"
subtitle: "Behind the Pretty Frames Of The Last Part Of Us"
---

![]({% link Game/TLPOU/Image/main.webp %})

___
- [介绍](#介绍)
- [准备](#准备)
  - [布料模拟](#布料模拟)
  - [GPU蒙皮](#gpu蒙皮)
  - [PreZ](#prez)
  - [ShadowPass](#shadowpass)
  - [半透明Depth](#半透明depth)

___

## <a id="介绍"></a>介绍

《最后生还者》又名《最后的着色器》、《显卡生还者》是由索尼旗下王牌工作室顽皮狗开发的一款3A级游戏。你将在cpu100%占用，爆显存，内存不足的情况下编译着色器，逐步发掘IGN10分的真相。

本篇只分析角色（笔记本3060带不动场景，只好进角色预览界面开角色最高画质看看，等啥时候换设备再说。）

![]({% link Game/TLPOU/Image/Ellie.jpg %} )
*基于该帧分析（构建着色器1%）*

角色展台界面使用了三个队列（Queue）进行异步计算，一个主图形队列（Graphic Queue）和两个计算队列（Compute Queue）。

![]({% link Game/TLPOU/Image/Queue.png %} )

图中的Command Queue2使用Computer Shader计算布料模拟，Command Queue4使用Computer Shader计算(Volume Based)Probe。可以看到，基本在两个Compute Queue完成后，Command Queue1才开始正式工作，这里主要关注Commad Queue1也就是实际绘制的内容。

<font size="1">（构建着色器1%）</font>

## <a id="准备"></a>准备

把BasePass之前的计算都算准备（吧），有GPU蒙皮+PreZ+阴影Pass。

### <a id="布料模拟"></a>布料模拟



<font size="1">（构建着色器1%）</font>

### <a id="GPU蒙皮"></a>GPU蒙皮

开始会有一大串的Dispatch操作（computer shader），该操作进行模型的蒙皮。

![]({% link Game/TLPOU/Image/GPUSkinEvent.jpg %} )

![]({% link Game/TLPOU/Image/GPUSkinShader.jpg %} )

注意的是，整个绘制流程中，模型数据都存在几个Buffer里，绘制时只是通过view description偏移起始位置，定义数据类型来位置采样Buffer，所以基本上抓帧软件都不能很好的直接看到模型。（之后开始画会细说）。

<font size="1">（构建着色器1%）</font>

### <a id="PreZ"></a>PreZ

蒙皮之后就是PreZ，在PreZ绘制的IA(Input Assembly)阶段:

![]({% link Game/TLPOU/Image/IA.png %} )

可以看到只有一个Index Buffer,我们看其他游戏绘制时的IA部分：

![]({% link Game/TLPOU/Image/IA1.png %} )

和明显的看到少了Input Layout部分的输入数据，也就是模型的数据，那模型数据在哪呢？我们看VS部分

![]({% link Game/TLPOU/Image/VS.png %} )

可以看到会比平常多出两个Buffer。就像之前提到的，在VS阶段会采样这个Buffer作为模型数据来绘制，IA阶段只提供顶点索引。至于为啥这样做：

![]({% link Game/TLPOU/Image/DrawInstance.png %} )

我猜测和DrawIndexedInstanced有关？（好像和啥GPU Driven Pipeline有关，阿巴阿巴）。

输出的深度PreZ结果：

![]({% link Game/TLPOU/Image/PreZ_Depth_Remap.png %} )

Stencil Output：

![]({% link Game/TLPOU/Image/PreZ_Stencil.png %} )

此外还有一个特殊的输出，值范围很大，和gl_PrimitiveID和Constant Buffer数据相关，可能是ID？

![]({% link Game/TLPOU/Image/PreZ_R.png %} )

### <a id="ShadowPass"></a>ShadowPass

这个没啥好说的，五个平行光，五个shadowmap，展示界面就是奢侈

<div calss="row">
    <div class="column5">
        <img src="{%link Game/TLPOU/Image/ShadowMap1.png%}">
    </div>
    <div class="column5">
        <img src="{%link Game/TLPOU/Image/ShadowMap2.png%}">
    </div>
    <div class="column5">
        <img src="{%link Game/TLPOU/Image/ShadowMap3.png%}">
    </div>
    <div class="column5">
        <img src="{%link Game/TLPOU/Image/ShadowMap4.png%}">    
    </div>
    <div class="column5">
        <img src="{%link Game/TLPOU/Image/ShadowMap5.png%}">
    </div>
</div>

其实还有一个Depth Only Pass，估计也是个平行光，但没画东西，估计是禁用了。

### <a id="半透明Depth"></a>半透明Depth

之后还会画半透明的Depth，像一些细碎的头发，眉毛，以及背包上的表的半透明玻璃罩。

![]({% link Game/TLPOU/Image/Trans_Depth.gif %} )

![]({% link Game/TLPOU/Image/trans_Depth.png %} )
*输入*
![]({% link Game/TLPOU/Image/trans_Depth2.png %} )
*输出*

之后就开始画BasePass了，下篇再见（不知道会不会咕咕咕）。

<font size="1">（构建着色器2%）</font>

--------------
结尾稍微补充下两个Computer Queue的细节。

布料模拟使用了六个Pass的Compute Shader，每个Pass使用的线程数为1x1x1。

Probe计算有三个Pass，VolProbeBufferWriteCache（1x1x1），VolProbeBufferReadCache（7x1x1）和VolumetricsTemporalCombineProbeCacheFroxelsScalar（7x5x32）。

Probe相关的细节在[顽皮狗官方的分享](https://www.naughtydog.com/blog/naughty_dog_at_siggraph_2020)上可以找到,主要在Lighting Technology of The Last of Us Part II上。

---------------
Reference：

[游戏中的全局光照(四) Lightmap、LightProbe和Irradiance Volume](https://zhuanlan.zhihu.com/p/265463655)

[Render Graph in DirectX 12](https://zhuanlan.zhihu.com/p/101318415)

[Probe-Based Global Illumination](https://zhuanlan.zhihu.com/p/350753497)

[官方研究博客](https://www.naughtydog.com/blog/naughty_dog_at_siggraph_2020)