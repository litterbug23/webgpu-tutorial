# 为什么WebGPU一定会取代WebGL？

**答：因为慢。**



<br>

#### 什么是WebGPU？

先说一下什么是 GPU ？

GPU 是单词 graphics processing unit 的缩写，中文通常会被翻译为：`图形处理器` 或 `图形处理单元`。

所以 WebGPU 从字面上可以简单理解为：让浏览器(Web)拥有访问和使用 GPU 的能力。



<br>

> 补充个基础知识：如何查看自己电脑是否有 GPU ？
>
> 1. 打开系统任务管理器
> 2. 点击 `性能` 面板
> 3. 在左侧如果能够找到 GPU 则表示你的电脑有 GPU
> 4. 若你的电脑本身就没有 GPU，那你搞不了 WebGPU 了...
>
> 对于 WebGPU 而言，无论是 集成显卡，还是 独显，都是可以的。



<br>

> 关于 GPU 的更多介绍，推荐你阅读 `0向往0` 的这篇文章，
>
> 深入GPU硬件架构及运行机制：https://www.cnblogs.com/timlly/p/11471507.html



<br>

> 本文以下很多知识点来源于 GDG devfest 2021大会上 因特尔 GPU 专家 邵嘉炜 的分享。
>
> WebGPU，下一代 Web 图形技术：https://www.bilibili.com/video/BV12i4y1d7KQ



<br>

**WebGPU 是一种新的 Web API，可用于在图形处理器(GPU)上执行渲染和计算操作。**



![](https://raw.githubusercontent.com/puxiao/webgpu-tutorial/main/imgs/webgpu_logo.jpg)



<br>

**WebGL的短板：API 标准太过陈旧**

WebGL 最新的版本 2.0 发布于 2017 年，该版本支持的 GPU 标准对应的是 Open ES 3.0，发布于 2012 年，而 Open ES 3.0 桌面端对应的标准为 Open GL 3.2，发布于 2009 年。

> 2009 年已经是 12 年前了。

在如此陈旧的标准下，根本无法充分利用到当前计算机中的性能优势：多核 CPU + 并行/通用计算 GPU 的优势。



<br>

> Open GL 归属于 Khronos Group 这个组织，这个组织就没打算继续发展优化 Open ES 和 WebGL，Khronos 目前主要想发展的是另外一套标准：Vulkan，算是 OpenGL 的下一代版本。

> 有一种阴谋论，说 WebGPU 替代 WebGL 原因之一是因为 WebGL 背后的 Open ES 版权归 Khronos，所以 W3C 组织才希望搞出属于自己的 WebGPU，一派胡言。
>
> WebGL 在渲染性能方面实在是没有跟上时代，这也是为什么 WebGL 网页 3D 实际上并没有大规模流行起来的原因。甚至学习 WebGL、Three.js 都被称为 `小众`。



<br>

> 补充：苹果浏览器 Safari 2021年 9 月才开始支持 WebGL 2.0。
>
> 为啥苹果浏览器对 WebGL 2.0 支持度这么晚？因为苹果觉得 WebGL 太慢，于是自己搞了一个 Metal 架构，将 WebGL 2.0 构建在自己的 Metal 架构之下，以便提升 WebGL 的性能。
>
> 实际上这个性能的提升是有限的，且仅适用于苹果浏览器，不适用于谷歌浏览器。



<br>

**天下分久必合 合久必分**

在 WebGL 不给力，且提升无望的情况下，各家各自造自己的轮子。

最终 微软、浏览器厂商、芯片厂商、W3C、Khronos 等多方达成共识：放弃 WebGL，开发全新的 Web 图形渲染 API ，就是 WebGPU。

> 浏览器厂商：苹果浏览器、谷歌浏览器、火狐浏览器等等主流浏览器
>
> 芯片厂商：英特尔、AMD等
>
> W3C：万维网联盟，Web 标准的制定者
>
> Khronos：OpenGL / Open ES 的版权方



<br>

**多方对WebGPU达成的共识：**

WebGPU 必须可以同时兼容 D3D12(微软)、Metal(苹果)、Vulkan(Khronos) 这 3 个底层框架。

1. 相同的 API 和 着色器语言

   > WebGL 对应的着色器语言为 GLSL(OpenGL Shading Language)

2. 减少 CPU 开销

   > 具体体现在 优化渲染管线

3. 更好支持 多线程

   > WebGL 渲染命令一旦调用，就会立即全局生效，同步执行，所以是单线程。
   >
   > 而 WebGPU 则将渲染命令拆分成 2 个步骤：录制命令(纯CPU操作) + GPU执行(多线程并行计算)

4. 支持现代 GPU 架构

5. 支持 GPU 通用计算

   > 关于 通用计算，这可能是 WebGPU 会爆发的另外一个点，不仅仅是用作 Web 3D 渲染，还为浏览器提供 GPU 通用计算的能力。

   > TensorFlow.js 有可能也会迎来爆发。

   > 此外 WebGPU 还支持 Shared Memory(共享内存)，简单来说可以定义一些变量让不同的进程都可以访问到该变量，极大提升 GPU 计算能力。
   >
   > 注：共享内存(Shared Memory) 和 全局变量 并不是同一回事。



<br>

**WebGPU结构体系图：**

你的程序(浏览器网页) > WebGPU API > 系统平台(OS) > { DirectX 12、Metal、Vulkan }

![](https://raw.githubusercontent.com/puxiao/webgpu-tutorial/main/imgs/webgpu_architecture_diagram.png)

> 图片来源：https://web.dev/gpu/

> 特别补充：
>
> 1. 上图中的 DirectX 12 就是 `D3D12` 。
> 2. WebGL 适用的是 DirectX 11，也就是 D3D11。



<br>

**WebGPU通过充分利用GPU并行计算的能力来提升渲染 3D 性能。**

关于 “并行计算” 的一个举例说明：

假设现在要计算 8 个数字的和，例如 2 + 4 + 3 + 6 + 8 + 1 + 7 + 9 等于多少，那么：

> 这里我们假设每 2 个数字求和所需时间为 1 毫秒 (我只是举例)

1. WebGL 是`单线`计算，它的步骤是：从左到右逐一计算，即 先计算前两个数之和(2 + 4 = 6)，然后将计算结果再与第 3 个数字求和(6 + 3 = 9)，然后再继续......

   那么这 8 个数字求和一共需要 7 毫秒。

2. WebGPU 则采用 `并行计算`，它的步骤是：将 8 个数字分成 4 组(每组 2 个数)，然后这 4 组 分别同时求和，得到 4 个数字，然后将这 4 个数字再拆分成 2 组(每组 2 个数字)，再次分别同时求和，将得到的 2 个数字进行最终的求和。

   那么这 8 个数字求和一共需要 3 毫秒。

   > 7 毫秒 VS 3 毫秒
   >
   > 上面使用 8 个数字求和仅仅是举例，实际中 3D 渲染所需的运算量非常大，可想而知性能差异会更大。

   > 你可以把 WebGL 对应的 CPU 单线计算看作是 1 位拥有超高心算能力的强者。
   >
   > 而 WebGPU 对应的 GPU 并行计算 则是 1 万名普通人。
   >
   > `1 位强者` VS  `1 万个普通人` ，强者再强也是没有胜算的。



<br>

总之，记住一句话：**WebGPU 是个好东西，它取代 WebGL 是必然的，属于 WebGPU 的时代即将到来**。



<br>

**WebGPU目前处于什么阶段？**

WebGPU 项目地址：https://github.com/gpuweb/gpuweb

| 阶段                           | 状态   |
| ------------------------------ | ------ |
| 创建解释器(草案)               | 完成   |
| 创建规范的初稿                 | 进行中 |
| 收集反馈并迭代设计             | 进行中 |
| 浏览器试用(内测)               | 进行中 |
| 全面启动(各大浏览器均开放支持) | 未开始 |

> 目前 谷歌浏览器(Chrome)、火狐(Firefox)、苹果浏览器(Safari) 的开发版均已开始支持 WebGPU。
>

> 补充：苹果浏览器原本自己搞的那套框架名字就叫 “WebGPU”，后来顾全大局将这个名字贡献给了 W3C，将自己的那个框架改名为 Web Metal。



<br>

**什么时候WebGPU会正式发布？**

目前网上并没有查到 WebGPU API  V1.0 正式发布时间，但是 Chrome 开发版 对 WebGPU 内测的声明中有这样一句话：“预计在 2022 年 5 月 18 日，大约 Chrome 101 版本后会停止试用。”

所以我猜测，大概率会在那个时候会正式发布 WebGPU 吧。



<br>

**WebGPU还有半年才会正式发布，现在就开始学？**

是的，我是这样认为的，我也准备这样做的。



<br>

下一篇，我会讲一下如何在各大浏览器上开启和调试 WebGPU。

