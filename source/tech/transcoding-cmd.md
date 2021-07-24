# 音视频转码与处理方法

## 有用的FFmpeg Filter

| Filter           | 功能                                                   |
| ----             | ----                                                   |
| bilateral        | 双边滤波，空间平滑时保持边缘                           |
| bm3d             | BM3D去噪算法                                           |
| blackdetect      | 黑帧时段检测                                           |
| blackframe       | 黑帧检测                                               |
| blend            | 混合2幅图像                                            |
| ciescope         | 显示图像色彩在CIE色度图上的分布                        |
| codecview        | 可视化某些codec的导出信息                              |
| colorbalance     | 色彩平衡，调整输入图像的基色(RGB)的相对强度            |
| colorcontrast    | 对比度调整，在RGB域上调整对比度                        |
| colorcorrect     | 白平衡，在YUV域上针对黑白色操作                        |
| colorchannelmixer| 组合色彩通道                                           |
| colorize         | 叠色，在图像上叠加某种色彩                             |
| colormatrix      | Matrix转换                                             |
| colorspace       | Matrix/Transfer/Primary转换                            |
| colortemperature | 色温调整                                               |
| crop             | 空间裁剪                                               |
| cropdetect       | 裁剪参数检测，检测四周黑边                             |
| curves           | 使用曲线调整色彩                                       |
| deband           | 去banding效应                                          |
| deblock          | 去块效应                                               |
| delogo           | 去logo                                                 |
| derain           | 去雨，基于CNN                                          |
| edgedetect       | 边缘检测，基于Canny算子                                |
| entropy          | 信息熵计算，基于灰阶直方图                             |
| eq               | 均衡化，可设置亮度、对比度、饱和度和调整伽马           |
| format           | 转换像素格式                                           |
| minterpolate     | 插帧，基于运动插值                                     |
| scdet            | 视频场景变化检测                                       |
| select           | 选择视频帧，可根据编码类型、帧号、pts、场、场景变化选择|

## 常见场景处理方法

FFmpeg Filtergraph示例
```
ffmpeg -i test.tif -lavfi "[0:0]sdr2hdr[hdr];[hdr]scale[rgb]" -an -v debug -map '[rgb]' hdr.tif
ffmpeg -i test.tif -i ref.tif -lavfi "[0:0]scale[yuv0];[yuv0]split[tst0][tst1];[1:0]scale[ref];[ref]split[ref0][ref1];[tst0][ref0]psnr=f=psnr.log;[tst1][ref1]ssim=f=ssim.log" -f null - > metric.log 2>&1
```

提取信息
```
mediainfo --Inform="Video;%Width%,%Height%,%FrameRate%,%BitRate%,%Duration%,%FrameCount%" input.mp4
mediainfo --Inform="Video;%colour_primaries%,%transfer_characteristics%,%matrix_coefficients%" input.mp4
magick identify -format "%wx%h %[colorspace] %[profile:icc]" input.tif
ffprobe -show_streams input.mp4
```

视频裁剪（时域）
```
ffmpeg -ss 01:00:00 -i input.mp4 -codec copy -t 06:00 output.mp4
```

视频拼接（时域）
```
MP4Box -cat 1.mp4 -cat 2.mp4 concat.mp4
ffmpeg -f concat -i list.txt -c copy concat.mp4
```
其中list.txt的内容为：
```
file ./1.mp4
file ./2.mp4
```

视频拼接（空域）
```
ffmpeg -i input1.mp4 -i input2.mp4 -lavfi [0:v][1:v]hstack output.mp4  # 空间上横向拼接
ffmpeg -i input1.mp4 -i input2.mp4 -lavfi [0:v][1:v]vstack output.mp4  # 空间上纵向拼接
```

视频缩放
```
ffmpeg -i input.mp4 -lavfi scale=w=1920:h=-2:force_original_aspect_ratio=decrease:flags=lanczos -c:v libx265 output.mp4
ffmpeg -i input.mp4 -lavfi scale=w=1920:h=1080:force_original_aspect_ratio=disable:force_divisible_by=2:flags=lanczos -c:v libx265 output.mp4
```

视频填充
```
ffmpeg -i input.mp4 -lavfi pad="iw:iw*9/16:0:(oh-ih)/2:black" -c:v libx265 output.mp4  # 上下填充两个黑边到16:9
ffmpeg -i input.mp4 -lavfi pad="ih*16/9:ih:(ow-iw)/2:0:black" -c:v libx265 output.mp4  # 左右填充两个黑边到16:9
```

添加静态LOGO
```
ffmpeg -i input.mp4 -i logo.png -lavfi "overlay=W-54-w:54" -c:a copy -c:v libx265 -x265-params crf=22 output.logo.mp4
```

音视频合并
```
MP4Box -add video_file#video -fps 25 -add audio_file#audio -new mp4_file
```

提取视频码流
```
MP4Box test.mp4 -raw 1 -out test.hvc  # 提取轨道1，不加-out默认生成test_track1.xxx
ffmpeg -i hevc.mp4 -an -vcodec copy -f hevc bitstream.265
ffmpeg -i h264.mp4 -an -vcodec copy -f h264 bitstream.264
ffmpeg -i in.mp4 -c:v copy -bsf hevc_mp4toannexb out.h265
ffmpeg -i in.mp4 -c:v copy -bsf h264_mp4toannexb out.h264
```

提取音频码流
```
ffmpeg -i test.mp4 -map 0:0 -c:a copy -vn out.ec3
```

视频抽帧
```
ffmpeg -i input.mp4 -vframes 10 -q:v 2 output_%03d.jpg  # better quality with smaller q, which locate in [1, 31]
ffmpeg -i input.mp4 -vf "select=eq(n\,34)+eq(n\,80)" image_%d.jpg  # number start from 0
```

图片格式转换
```
# TIF -> JPG
magick convert input.tif +profile * -delete 1--1 -profile sRGB_ICC_v4_Appearance.icc -resize x480 -quality 92 output.jpg
+profile *   : 移除所有内嵌profile
-profile     : 嵌入profile
-delete 1--1 : 只保留第0个输出文件，删除第1到-1（最后一个）输出文件，主要针对输入图片multi-dirtory的情况
-resize x480 : 高度缩放到480，宽度以保持宽高比为准

# TIF -> EXR
```

图片拼接
```
magick convert in1.jpg in5.jpg +append out-1plus5.jpg # 横向拼接
magick convert in1.jpg in5.jpg -append out-1plus5.jpg # 纵向拼接
magick montage -mode concatenate -tile x1 in*.jpg out.jpg    # 横向拼接
magick montage -mode concatenate -tile 1x in*.jpg out.jpg    # 纵向拼接
```

混音(多声道向下混成2声道)
```
ffmpeg -i db.wav -c:a pcm_s24le -ac 2 db.ac2.wav
ffmpeg -i db.wav -af "pan=stereo | FL < FL + 0.5*FC + 0.6*BL + 0.6*SL | FR < FR + 0.5*FC + 0.6*BR + 0.6*SR" db_stereo_DIY.wav
ffmpeg -i "xxxx" -map 0:v -vcodec libx264 -b:v 15000k -lavfi "[0:1][0:2][0:3][0:4][0:5][0:6[0:7][0:8] amerge=inputs=8" -acodec libfdk_aac -b:a 384k -strict -2 -y "xxxx"
```

增加静音轨(FFmpeg版本需要在v4.1.0之上)
```
ffmpeg -i video.mp4 -an -f lavfi -i anullsrc=channel_layout=stereo:sample_rate=44100 -c:v copy -shortest -f mp4 video.mute.mp4
```

计算PSNR, SSIM和VMAF
```
ffmpeg -i test.mp4 -i ref.mxf -lavfi "psnr=f=psnr.log;[0:v][1:v]ssim=f=ssim.log" -f null - > metric.log 2>&1
ffmpeg -i test.mp4 -i ref.mp4 -lavfi libvmaf="model_path=release/model/vmaf_v0.6.1.pkl:log_path=vmaf.json:log_fmt=json" -f null - > metric.log 2>&1
```

HDR退化为SDR (编译时需要开启--enable-libzimg)
```
ffmpeg -i HDR.mp4 -vf zscale=transfer=linear,tonemap=reinhard,zscale=transfer=bt709,format=yuv420p -c:v libx265 -x265-params crf=21 -movflags faststart SDR.reinhard.mp4
```
