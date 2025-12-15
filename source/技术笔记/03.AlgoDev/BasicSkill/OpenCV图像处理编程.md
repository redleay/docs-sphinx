
# cv2.resize()

注意所采用的插值算法，下采样使用`cv2.INTER_CUBIC`、`cv2.INTER_LINEAR`、`cv2.INTER_LANCZOS4`可能会导致输出的图像存在黑色噪点，
建议使用`cv2.INTER_AREA`

# cv2.createCLAHE()

使用方法
```
clahe = cv2.createCLAHE(clipLimit=3.0, tileGridSize=(8, 8))
enhanced = clahe.apply(gray)
```

# cv2.Threshold

# cv2.adaptiveThreshold

# cv2.GaussianBlur()

# cv2.Canny()

[Canny边缘检测算法](https://zhuanlan.zhihu.com/p/99959996)

# cv2.erode()
腐蚀

# cv2.dilate()
膨胀

# cv2.morphologyEx()

kernel = cv2.getStructuringElement(cv2.MORPH_ELLIPSE, (3, 3))
closed_edges = cv2.morphologyEx(edges_combined, cv2.MORPH_CLOSE, kernel, iterations=2)

# cv2.findContours()

寻找轮廓

> 参数1：源图像
>
> 参数2：轮廓的检索方式，这篇文章主要讲解这个参数
>
> > cv.RETR_LIST：最简单的一种寻找方式，它不建立轮廓间的子属关系，也就是所有轮廓都属于同一层级。这样，hierarchy中的后两个值[First Child, Parent]都为-1
> >
> > cv.RETR_TREE：就是之前我们一直在使用的方式，它会完整建立轮廓的层级从属关系
> >
> > cv.RETR_EXTERNAL：这种方式只寻找最高层级的轮廓，也就是它只会找到前面我们所说的3条0级轮廓
> >
> > cv.RETR_CCOMP：把所有的轮廓只分为2个层级，不是外层的就是里层的
>
> 参数3：一般用 `cv.CHAIN_APPROX_SIMPLE`，就表示用尽可能少的像素点表示轮廓
>
> 参数4 contours：图像轮廓坐标，是一个链表
>
> 参数5 hierarchy：[Next, Previous, First Child, Parent]，轮廓层级的链表，具体含义如下：
>
> > Next：与当前轮廓处于同一层级的下一条轮廓
> >
> > Previous：与当前轮廓处于同一层级的上一条轮廓
> >
> > First Child：当前轮廓的第一条子轮廓
> >
> > Parent：当前轮廓的父轮廓

参考：[cv2.findContours()函数详解](https://www.cnblogs.com/wojianxin/p/12602490.html)

# cv2.contourArea()

计算轮廓面积（使用格林公式）

# cv2.arcLength()

计算轮廓周长，计算时使用轮廓的像素的中心点计算实际长度

# cv2.PCACompute

```
mean, eigenvectors = cv2.PCACompute
```

参数如下：

> data：包含每个数据点的数据矩阵，作为行向量或列向量。如果我们的数据由1000个图像组成，并且每个图像是30k长的行向量，则数据矩阵将为30k×1000。
>
> mean：数据的平均值。如果数据矩阵中的每个数据点都是30k长的行向量，则均值也将是相同大小的向量。此参数是可选的，如果未提供，则在内部计算。
>
> flags：它可以取值DATA_AS_ROW或DATA_AS_COL，指示数据矩阵中的点是沿着行还是沿着列排列。在我们共享的代码中，我们将其安排为行向量。
>
> maxComponents：确定主成分的个数。主成分的最大数量通常是两个值中较小的一个：原始数据的维度（在我们的例子中是30k）或数据点的数量（例如上例中的1000）。但是，我们可以通过设置此参数来明确确定我们想要计算的最大主成分数
