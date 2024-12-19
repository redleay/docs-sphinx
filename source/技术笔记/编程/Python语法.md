# Python语法


替换：数字替换为星号*

```
new = re.sub("[0-9]", "*", s)
```

## 数据类型处理

List转Set: `set(listA)`

List转Dict: `dict.fromkeys(listA)`

List去重: `list(set(listA))`

List追加合并: `listC = listA + listB`

List连接: `chain(bars_576p, bars_720p, bars_1080p)`

## 迭代式

单List循环: `[v for v in values]`

双List循环: `[w for v in values for w in v.get("width")]`

单List循环返回索引值: `[i for i,v in enumerate(vmaf_list_wesee) if v < 82.0]`

多List循环返回特定值: `[b for w,b in zip(width_list, bitrate_list) if w == 720]`

dict.keys()
dict.values()
for key,value in dict.items()

## 数学计算

```
round()         # 四舍五入
math.floor()    # 向下取整
math.ceil()     # 向上取整
```

## Numpy

手动定义np.array
```
coef = np.array([[1688, 683], [2146, 2951]])
```

numpy.array：可直接通过`+-*/`实现逐个元素的加减乘除运算

List转numpy.array：`np.array(listA)`

numpy.array转List: `listA = numpyarrayA.tolist()`

二维list转一维：`np.array(matrix).flatten().tolist()`

内置索引和赋值: `rgb[rgb < 0] = 0`

条件选择和处理：`a = np.where((b > 0) & (b < 3), 1, 7)`

条件选择和处理：`a = np.select([(b > 0) & (b < 3), (b > 6) & (b < 8)], [1, 7])`

计算分位点：`numpy.percentile(list, pencenttile)`
四舍五入：`numpy.around(list)`

numpy.bincount(x, /, weights=None, minlength=0)
numpy.histogram(a, bins=10, range=None, density=None, weights=None)
cv.calcHist()

https://numpy.org/doc/2.0/reference/generated/numpy.bincount.html

```
np.arange(low, high, step)
img = np.power(img, m)
img = np.multiply(img, c3)
lms = np.dot(rgb, coef)
rgb = np.clip(rgb, 0, 1)
mask = np.ones((1, 1, 3), dtype=np.uint16)
mask = np.zeros((1, 1, 3), dtype=np.uint16)
```

[教程](https://www.runoob.com/numpy/numpy-tutorial.html)

## 数学统计

```
avg,std = cv2.meanStdDev(img)：计算图像各通道的平均值和标准差
np.average()：根据在另一个数组中给出的各自的权重计算数组中元素的加权平均值
np.mean()：数组中元素的算术平均值，可按轴计算
np.median()：中位数
np.amin()：计算数组中的元素沿指定轴的最小值
np.amax()：计算数组中的元素沿指定轴的最大值
np.ptp()：计算数组中元素最大值与最小值的差
```

## 图像处理

读入和写出图像
```
img = cv2.imread(input, cv2.IMREAD_UNCHANGED)  # [H, W, C]
cv2.imwrite(output, img)
```

色彩空间转换：RGB/GBR通道转换
```
rgb = cv2.cvtColor(bgr, cv2.COLOR_BGR2RGB)
bgr = cv2.cvtColor(rgb, cv2.COLOR_RGB2BGR)
```

分离和合并通道
```
b,g,r = cv.split(img)
img = cv.merge([b,g,r])
```

## 日志检查

```
assert CONDITION, MESSAGE
```

## matplotlib绘图

绘制直方图
```
def get_ticks(data, batch):
    bias    = batch / 2
    low     = math.floor(min(data) / batch) * batch - bias
    high    = math.ceil (max(data) / batch) * batch + batch
    bins    = np.arange(low, high, batch)
    ticks   = bins + bias
    return bins,ticks

plt.figure(figsize=(16,9))
bins,ticks = get_ticks(bitrate_list, 100)
bars, _, _ = plt.hist(bitrate_list, bins, align='mid', rwidth=0.9)
y_max = bars.max()
plt.vlines(bitrate_list.mean(), 0, y_max, label="Average", colors="r", linestyles="dashed")
plt.xticks(ticks, rotation=-45)
plt.yticks(np.arange(0, y_max+1))
plt.title ("Bitrate Distribution")
plt.xlabel("Bitrate(kbps)")
plt.ylabel("Number")
plt.legend()
plt.savefig("Bitrate.png", bbox_inches='tight') # tight output
```

## 其他

修改Linux系统语言编码，解决python获取Linux命令输出的中文字符乱码问题
```
export LC_ALL="en_US.utf8"
```

递归列出目录下所有mp4文件
```
glob.glob('./test/**/*.mp4', recursive=True)
```

检测文件名字符串 name 是否匹配模式字符串 pat
```
fnmatch.fnmatch(name, pat)      # 大小写不敏感，pat = '*.mp4'
fnmatch.fnmatchcase(name, pat)  # 大小写敏感
```

查找匹配 pat 的文件列表，基于 iterable names 中匹配模式 pat 的元素构造一个列表
```
fnmatch.filter(names, pat)
```

os.listdir(path)


list原地排序
filelist.sort()

