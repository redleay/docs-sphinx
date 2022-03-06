# 音视频多媒体笔记

HLS:
HTTP Live Streaming, 索引文件为m3u8，数据文件原为ts，目前也支持fmp4，原来只支持H.264，后增加支持H.265

DASH:
Dynamic Adaptive Streaming over HTTP, 索引文件为mpd，数据文件支持fmp4和ts，对编码格式无限制

mpd : Media Presentation Description
fmp4: Fragmented MP4

HLS with fmp4:
Apple增加HLS对fmp4的支持后，由于DASH也支持fmp4，HLS和DASH可以共用fmp4文件，不再需要分别存储两份视频数据，节省了存储空间。

Fmp4与mp4区别:
mp4只包含一个巨大的mdat box，不方便seek码流位置和切换流。fmp4包含一个sidx box和一系列的moof+mdat box，一般使用固定的时间段来划分fragment，方便关键帧对齐和seek码流位置，更加适合于切换不同码流和清晰度的流的场景

ISO/IEC 14496-15标准文档针对HEVC定义了hvc1与hev1两种封装格式，两者的区别在于：
hvc1: VPS/SPS/PPS只能出现在码流最前面
hev1: VPS/SPS/PPS可以出现在码流中间

声道正常顺序
```
L R C LFE Ls Rs          # 5.1声道
L R C LFE BL BR          # 5.1声道
L R C LFE Ls Rs Lrs Rrs  # 7.1声道
```
多声道音轨名称对应关系
```
L = (Front) Left
R = (Front) Right
C = Center
LFE = Low Frequency Effects
Ls = Left Surround (Rear Left)
Rs = Right Surround (Rear Right)
Lrs = Left Rear Surround
Rrs = Right Rear Surround

Lc = Left Center
Rc = Right Center
Lm = Left Mid
Rm = Right Mid
S = Surround (Rear Center)
```

IRE：是一个用于测量模拟复合视频信号强度（亮度）的单位，0IRE为黑电平，白电平为+100IRE。一个IRE相当于1/140V，NTSC视频的1IRE为7.14mV，而在其他视频系统中，1IRE相当于7mV。IRE其实是Institute of Radio Engineers（无线电工程师协会）的缩写，IRE就是由该协会所制定的单位

对比度：对比度（Contrast）代表在灰阶范围内，最高输出亮度和最低输出亮度的比值。比值越高，证明监视器细节表现越真实，图像的清晰度、灰阶层次表现越好。(OLED黑场几乎是全黑，普通仪器无法正确读到亮度，对比度值可能为无限大。)

## 电子书格式
常见的电子书后缀有epub、mobi、azw、azw3。epub格式基于国际通行的开放标准ePub，亚马逊先后以ePub为基础，开发了KF6、KF7、KF8标准。
mobi格式一般为KF6标准，又称mobi6
azw格式和mobi7格式都基于KF7标准，区别在于azw支持DRM，而mobi7不支持
azw3格式和mobii格式都基于KF8标准，区别在于azw3支持DRM，而mobi8不支持
