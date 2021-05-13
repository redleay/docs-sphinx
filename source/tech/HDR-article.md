# HDR和色彩相关文章

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

## 编码

[HDR关键技术：HEVC/H.265编码优化 - 云+社区 - 腾讯云](https://cloud.tencent.com/developer/article/1180918)

## 色彩表示

人眼中有三种分别对L、M、S三种波长敏感的锥状细胞，由三种锥状细胞得到的三刺激值可构成LMS色彩空间，但直接使用LMS三刺激值得到的色度图很大一部分落在第二象限，原因是红色的刺激值存在负值。

CIE XYZ表示的是由XYZ三刺激值绘制的色彩空间，XYZ三刺激值由LMS三刺激值转换而来，重新选择了三种基色，将红色刺激值全部转换为非负值，因此XYZ得到的色度图全部落在第一象限。

CIE XYZ的XZ代表色度，Y代表亮度。但需要注意的是，Y代表的是相对亮度而不是绝对亮度，即在色彩的辐射总能量相同的情况下，该色彩相对其他色彩的亮度。人眼感觉色度图的绿色区域较亮，蓝色部分较暗，原因就是色度图是从X + Y + Z = 1的单位光面上截出来的，在总的光辐射能量相同的情况下，人眼对绿色敏感因此觉得更亮，但这并不意味着绿色比所有红色都亮。

色彩的实际亮度取决于Y，XYZ、xyY和YUV中的Y完全等同，是由RGB加权计算得出的，加权权重由RGB的基色坐标和白点确定，具体可参考RGB2XYZ的计算过程。

CIE xyY由CIE XYZ派生而来，xyY的Y与XYZ的Y完全等同，xy由XY归一化得来，平常常见的CIE色度图其实是xyY对应的色度图，坐标已经进行了归一化处理。

[色彩空间表示与转换 - 知乎](https://zhuanlan.zhihu.com/p/24281841)

[总结各种RGB转YUV的转换公式 - 郑建宏 - 博客园](https://www.cnblogs.com/zhengjianhong/p/7872459.html)

[A Beginner’s Guide to (CIE) Colorimetry – Color and Imaging – Medium](https://medium.com/hipster-color-science/a-beginners-guide-to-colorimetry-401f1830b65a)

[Useful Color Equations](http://www.brucelindbloom.com/index.html?Math.html)

[RGB/XYZ Matrices](http://www.brucelindbloom.com/index.html?Eqn_RGB_XYZ_Matrix.html)

[RGB and YUV Color Space Conversion](https://www.vocal.com/video/rgb-and-yuv-color-space-conversion/)

[CVRL main](http://www.cvrl.org/)

[CVRL main](http://cvrl.ioo.ucl.ac.uk/)

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
