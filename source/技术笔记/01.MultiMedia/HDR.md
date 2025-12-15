# HDR

## 综述

[HDR技术浅析 - 知乎](https://zhuanlan.zhihu.com/p/31768560)  
[浅谈HDR视频技术 - 知乎](https://zhuanlan.zhihu.com/p/42340278)  
[一文解读HDR原理及不同的标准 - 知乎](https://zhuanlan.zhihu.com/p/35406753)  
[浅谈HDR视频格式 - 知乎](https://zhuanlan.zhihu.com/p/41787860)  
[HDR关键技术：主要标准介绍 - 云+社区 - 腾讯云](https://cloud.tencent.com/developer/article/1194060)  
[HDR标准详解：HDR10、Dolby Vision、HDR10+哪家强？ZNDS资讯](https://n.znds.com/article/29646.html)  
[What is HDR10+? - CNET](https://www.cnet.com/news/what-is-hdr10/)  

## 光电转换

[HDR详解 - 伽玛曲线](https://www.eizo.com.cn/global/library/management/ins-and-outs-of-hdr/index2.html)  
[UHDTV - HDR, HLG and WCG](https://www.lightillusion.com/uhdtv.html)  
[HDR Video Part 3: HDR Video Terms Explained — Mystery Box](https://www.mysterybox.us/blog/2016/10/19/hdr-video-part-3-hdr-video-terms-explained)  
[HDR视频生态圈追踪 - netease_im的博客 - CSDN博客](https://blog.csdn.net/netease_im/article/details/86583136)  
[为什么人的很多感觉都是对数的，例如视觉、听觉？ - 知乎](https://www.zhihu.com/question/24733163/answer/349614347)  
[gammacorrection](http://www.13thmonkey.org/~boris/gammacorrection/)  
下文讨论了Apple系统的QuickTime Gamma Shift问题的由来和可能的应对办法：
[Why are videos washed out on the Mac?](https://blog.dominey.photography/2021/01/24/why-are-videos-washed-out-on-the-mac-exploring-quicktime-gamma-shift/)

## 编码

[HDR关键技术：HEVC/H.265编码优化 - 云+社区 - 腾讯云](https://cloud.tencent.com/developer/article/1180918)  

## 视频封装

根据[High Dynamic Range Metadata For Apple Devices (Preliminary)](https://developer.apple.com/av-foundation/High-Dynamic-Range-Metadata-for-Apple-Devices.pdf)，
- HDR10的Mastering Display和Content Light信息优先存放于mdcv/clli/colr中，与hvcC并列，否则则应该存放于hvcC中
- HDR10的色彩描述信息应该存放于colr中

## 图像格式

### RGBE格式

又称为`Radiance HDR`、`HDRI`，每个像素使用4个byte表示，RGB通道分别使用1个byte表示尾数，同时共用1个byte作为指数，故称为RGBE。3个通道的尾数和指数结合后遵循IEEE半精度浮点数格式。

基于RGBE格式，演变出了一些其他变种，如`RGBM`和`RGBD`，主要是改变有效数字和指数的有效位数、指数部分使用除法代替乘法。

根据 [标准文件](https://radsite.lbl.gov/radiance/refer/filefmts.pdf)第30页，RGBE中会记录色域坐标，并且根据第33页，针对RGB数据定义了数据的物理单位为`watt/steradian/sq.meter`，因此可以转换为实际的物理亮度和色彩。

RGBE文件中记录的重点信息如下：
- EXPOSURE：1个浮点数，用于对文件的所有像素的亮度值进行调整
- COLORCORR：3个浮点数，对应于3个通道，用于对文件的所有像素的对应通道亮度值进行调整
- PIXASPECT: 像素的宽高比，即PAR
- PRIMARIES: RGB三原色和白点在CIE色度图中的坐标，即色域
- 分辨率

[Wikipedia](https://en.wikipedia.org/wiki/RGBE_image_format)
[官方网页](https://radsite.lbl.gov/radiance/HOME.html)
[标准文件](https://radsite.lbl.gov/radiance/refer/filefmts.pdf)
[作者个人网页](https://radsite.lbl.gov/radiance/contributors/GregWard.html)

2024年9月16日，Apple发布了新的macOS 15 Sequoia系统，Preview支持显示`HDRI`格式，参考[View PDFs and images in Preview on Mac](https://support.apple.com/en-ie/guide/preview/prvw11470/mac)。

### EXR

全称是OpenEXR，支持16-bit Float, 32-bit Float, 32-bit Int三种数据类型。

根据[EXRTechnicalIntroduction.pdf](https://people.computing.clemson.edu/~dhouse/courses/404/papers/EXRTechnicalIntroduction.pdf)第18页，
EXR标准中支持写入色域坐标信息，如果未写入，则默认为BT.709色域。

根据[EXRTechnicalIntroduction.pdf](https://people.computing.clemson.edu/~dhouse/courses/404/papers/EXRTechnicalIntroduction.pdf)第18页，
亮度数值0.18定义为18%灰度值，即SDR标准下的中灰亮度，也即对应拍摄中的1个stop，数值1.0代表100%反射物的亮度值。
归一化后的16-bit Float可表示30个stop，每个stop拥有1024个step。

根据DeepSeek-R1模型的回答，EXR的头部元数据中存在`whiteLuminance`元数据，用于定义物理映射关系。

### JPEG XT


## 后期制作
[一文掌握达芬奇最重要的节点：串行、并行和外部节点](http://www.cnrft.com/news/show/1466/)
[达芬奇的并行节点/串行节点/层节点的用途和三者的区别](https://www.newvfx.com/forums/topic/30515)

## 其他

[View of Labels](https://registry.smpte-ra.org/view/published/labels_view.html)  
[关于华为P30 Pro RYYB传感器的猜测。顺带整理数码传感器的基本原理](https://www.weibo.com/ttarticle/p/show?id=2309404354295012142481)  
[standards / HDRTools · GitLab](https://gitlab.com/standards/HDRTools/)  
[高动态图像(HDR)数据库整理 - 简书](https://www.jianshu.com/p/c3a85cbbc4ca)  
[MIT-Adobe FiveK dataset](http://data.csail.mit.edu/graphics/fivek/)  
[HDR-VDP](http://resources.mpi-inf.mpg.de/hdr/vdp/)  
[hdrvdp](http://hdrvdp.sourceforge.net/wiki/)  
[采用增强网络的色彩增强算法1801 - 子石的博客 - CSDN博客](https://blog.csdn.net/u011501411/article/details/82349616)  
[HDR中的Tone Mapping](https://blog.csdn.net/ys5773477/article/details/53492425)  
