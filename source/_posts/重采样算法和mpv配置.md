---
title: 重采样算法和 mpv 配置
date: 2024-08-18 23:00:00
# categories: 全新体验
tags:
sticky: 1
#cover
comments: true
toc: true
author: DaydreamWarrior
---

因为想好好看番，同时屏幕不是 1080p 默认的双线性缩放质量很差所以被迫学习了下。

视频相关知识是看的 vcb-s 的[教程](https://github.com/vcb-s/guides)，比如压缩原理和视频瑕疵。

播放器先参考 vcb-s 的 [mpv 指南](https://vcb-s.com/archives/7594)，至少先把 mpv 下载下来，我用的是 iina。

重采样这些是看的 [imagemagick](https://www.imagemagick.org/Usage/filter/)。

重采样滤镜的[对比](https://artoriuz.github.io/blog/imagemagick_resampling.html)，我觉得只能反应谁锐，PSNR 和 SSIM 都比较神秘。

~~上面这些全都是前置知识，不看的话会看不懂我在说什么。~~

重采样大概就是缩放要么做插值，先对列做然后对行做（反之的结果是一样的），比如线性 `bilinear` 或者三次拟合 `cubic` 和 `spline`，还有 Mitchell-Netravali 系列的。或者就是卷积，比如乘上 $\text{sinc}$。

插值就是卷积，`bilinear` 和 `triangle` 是相同的。

满足奈奎斯特采样定律的话，$\text{sinc}$ 有可能还原出原本的曲线，$\text{sinc}$ 是理想低通滤波器。

$\text{sinc}$ 也不能直接做一整行的，不仅是性能还因为 ringing，找最近的若干个采样点做。

直接乘 sinc 会产生很强的 ringing，做了傅立叶变换之后就看得到，不加窗的旁瓣能量很大，虽然主瓣最窄也就是最锐利。[窗函数](https://zh.wikipedia.org/wiki/%E7%AA%97%E5%87%BD%E6%95%B0) 的旁瓣越小 ringing 越小，主瓣越宽越模糊。

所以要加窗，sinc windowed-sinc 就是 `lanczos` 算法，一般是 3tap。

也可以不行列然后做，ewa 相关算法就是直接采样圆内的点，用的是的 $\text{jinc}$，和 sinc 强相关，二维的理想低通滤波器。jinc windowed-jinc 就是 `ewa_lanczos`。jinc windowed-hann 就是 `ewa_hanning`。这个无法保留像素哈希模式，因为正交的贡献和对角的不一样，所以可以消除锯齿等。

也许会好奇 `ewa_lanczos` 的距离为什么是 $3.2383154841662362$，因为这是 $\text{jinc}$ 第三个 $0$ 的位置。

`lanczos` `spline36` `ewa_lanczos` 和其变体都被认为是比较好的算法，前两适合实拍后者适合 anime。

锐的程度

`ewa_lanczos4sharpest` > `lanczos` > `spline36` > `ewa_lanczossharp` > `ewa_hanning`

抗锯齿（和哈希模式排名一样，就是保留像素级别的形状）

`ewa_lanczossharp` > `ewa_hanning` > `ewa_lanczos4sharpest` > `lanczos`

最小 ringing

`ewa_hanning` > `ewa_lanczossharp` > `spline36` > `lanczos`

我用的是 `ewa_hanning`，因为 $<2$ 的缩放比例的锐度差距不太明显，同时 ringing 和锯齿都非常讨厌。

mpv 默认是 `lanczos`，`high-quality` 是 `ewa_lanczossharp`，两个的 ringing 都很重又不开 anti ringing，很魔怔。

$\geq 2$ 的缩放比例 RAVU FSRCNNX 和 anime4k 绝对会更好，虽然 anime4k 放在这里，但是它还有线条重构之类的东西。a4k 看着非常好，但是会把一些该模糊的东西也变清晰，比如制作省钱画崩的脸和路人的背景之类的，再加上我对 ai 之类的东西过敏所以没用。

视频还有色度上采样，可以参考 vcb-s 的那个教程，色度 chroma 的分辨率是亮度 LUMA 的一半。

我用的 KrigBilateral，大概是结合亮度信息作 krig 插值，效果几乎是最好的，但是人眼对色度不敏感，所以我带有心理安慰的成分。vcb-s 建议是 `catmull_rom`。mpv 默认的高质量非常魔怔，用的和上采样一样也是 `ewa_lanczossharp`。

我还是把 deband 打开了的，这也是 vcb-s 的建议。下采样我用的 `spline36`。

所以我的配置是
```
scale = ewa_hanning
dscale = spline36
dither-depth = 10
deband = yes
glsl-shaders = "~~/shaders/KrigBilateral.glsl"
blend-subtitles = video
```

这样的配置对于我性能也足够，看 live 和 anime 都比较通用，总之我现在比较满意，也绝对不再折腾了。

更多配置可以参考[官方手册](https://github.com/mpv-player/mpv/blob/master/DOCS/man/options.rst)。高质量的讨论可以参考升级 high-quality 默认的 [issue](https://github.com/mpv-player/mpv/pull/12384)。