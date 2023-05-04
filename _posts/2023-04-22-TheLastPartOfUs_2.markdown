---
layout: post
title:  "The Last Part Of Us(Part II)"
subtitle: "Behind the Pretty Frames Of The Last Part Of Us"
---

![Zelda]({% link Game/TLPOU/Image/main.webp %})

___
- [介绍](#介绍)
- [BasePass](#basepass)

___

## <a id="介绍"></a>介绍

<font size="2">《最后生还者》又名《最后的着色器》、《显卡生还者》是由索尼旗下王牌工作室顽皮狗开发的一款3A级游戏。你将在cpu100%占用，爆显存，内存不足的情况下编译着色器，逐步发掘IGN10分的真相。</font>

本篇进入BasePass部分。

![Zelda]({% link Game/TLPOU/Image/Ellie.jpg %} )
*基于该帧分析*

<font size="1">（构建着色器2%）</font>

## <a id="BasePass"></a>BasePass

进入BasePass部分，首先能看到DrawCall都作为ExecuteIndirect的子命令被调用（估计又是啥优化操作，咱也不知道）。

![Zelda]({% link Game/TLPOU/Image/BasePassCommand.png %} )

可以看到，有许多DrawIndexedInstanced命令里顶点数都是0，也就是啥也没画，不知道啥情况。

在BasePass部分，除去Depth和Stencil，输出了七张Buffer，有几张很土豪的使用了单通道16位的Buffer存储数据。

<div calss="row">
    <div class="column5">
        <img src="{%link Game/TLPOU/Image/RTV0.png%}">
        <em>RTV0(R16G16B16)</em>
    </div>
    <div class="column5">
        <img src="{%link Game/TLPOU/Image/RTV0_A.png%}">
        <em>RTV0(A16)</em>
    </div>
    <div class="column5">
        <img src="{%link Game/TLPOU/Image/RTV1.png%}">
        <em>RTV1(R16G16B16)</em>
    </div>
    <div class="column5">
        <img src="{%link Game/TLPOU/Image/RTV1_A.png%}">
        <em>RTV1(A16)</em> 
    </div>
    <div class="column5">
        <img src="{%link Game/TLPOU/Image/RTV2.png%}">
        <em>RTV2(R8)</em>
    </div>
    <div class="column5">
        <img src="{%link Game/TLPOU/Image/RTV3.png%}">
        <em>RTV3(R16B16)</em>
    </div>
    <div class="column5">
        <img src="{%link Game/TLPOU/Image/RTV4.png%}">
        <em>RTV4(R11G11B10)</em>
    </div>
    <div class="column5">
        <img src="{%link Game/TLPOU/Image/RTV5.png%}">
        <em>RTV5(R11G11B10)</em>
    </div>
    <div class="column5">
        <img src="{%link Game/TLPOU/Image/RTV6.png%}">
        <em>RTV6(R16G16B16)</em>
    </div>
    <div class="column5">
        <img src="{%link Game/TLPOU/Image/RTV6_A.png%}">
        <em>RTV6(A16)</em>
    </div>
</div>

需要注意的是，为了方便显示，我将Buffer内的值映射到了0-1的范围。可以肯定的的是，在这些Buffer内数据进行了压缩，单通道内往往通过位操作存储着两个甚至三个属性值的位数据。

在绘制完基础不透明物体后，会有两个额外的Pass处理眼角和口腔，这部分顽皮狗在Siggraph 2020上做了分享，不过当时是关于最后生还者II，重制版在22年9月出的，用了II的技术很合理。主要是通过Decal的方式模糊眼角法线更好的过度眼球和眼睑做泪线效果（在经典的角色渲染分享Next-Generation Character Rendering中就有提到屏幕空间折射的模糊处理），不过看到口腔也会画属实没想到。当然，口腔部分也可能只是用了Decal做其他的处理。数据经过了重压缩，所以看不出来有啥区别，这边只展示下绘制的网格（黑色）。（ps：乔尔居然是没有嘴唇部分Decal处理的，截这块的时候笔记本风扇呼呼转，截了我两次，两次都是有渲染bug，头发啥的显示有问题）

![Zelda]({% link Game/TLPOU/Image/Decal_Eye.png %} )
*眼角环绕一小圈*

![Zelda]({% link Game/TLPOU/Image/Decal_Tooth.png %} )
*像是牙齿根部*

![Zelda]({% link Game/TLPOU/Image/Decal_Tooth1.png %} )
*基本可以确认是牙龈（大概是这个部位）*

之后还有两次奇怪的绘制，其函数入口为"TestEnemyVisibleInfo"，应该是和敌人绘制相关，跳过。

两次ComputerShader计算焦距，其CS入口点为"Cs_GetFocusPositionCameraDistance"和"CS_Collect8x8FocusDistanceBuffer"。

之后做深度的降采样。

延迟灯光剔除

高光IBL

TAA深度

应用球谐

SSDO,SSAO

遮蔽阴影

SSS

DefferLighting

前向物体绘制