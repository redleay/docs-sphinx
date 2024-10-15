# Python语法

## 数据类型处理
List转Set: `set(listA)`

List转Dict: `dict.fromkeys(listA)`

List去重: `list(set(listA)`

List追加合并: `listC = listA + listB`

List连接: `chain(bars_576p, bars_720p, bars_1080p)`

## 迭代式

单List循环: `[v for v in values]`

双List循环: `[w for v in values for w in v.get("width")]`

单List循环返回索引值: `[i for i,v in enumerate(vmaf_list_wesee) if v < 82.0]`

多List循环返回特定值: `[b for w,b in zip(width_list, bitrate_list) if w == 720]`

## 数学计算

```
math.floor()
math.ceil ()
```

## Numpy
手动定义np.array
```
coef = np.array([[1688, 683], [2146, 2951]])
```

numpy.array：可直接通过`+-*/`实现逐个元素的加减乘除运算

List转numpy.array：`np.array(listA)`

numpy.array转List: `listA = numpyarrayA.tolist()`

内置索引和赋值: `rgb[rgb < 0] = 0`

条件选择和处理：`a = np.where((b > 0) & (b < 3), 1, 7)`

条件选择和处理：`a = np.select([(b > 0) & (b < 3), (b > 6) & (b < 8)], [1, 7])`

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

## 图像处理

读入和写出图像
```
img = cv2.imread(input, cv2.IMREAD_UNCHANGED)  # [H, W, C]
cv2.imwrite(output, img)
```

RGB和GBR通道顺序转换
```
rgb = cv2.cvtColor(bgr, cv2.COLOR_BGR2RGB)
bgr = cv2.cvtColor(rgb, cv2.COLOR_RGB2BGR)
```

计算图像各通道的平均值和标准差
```
avg,std = cv2.meanStdDev(img)
np.average()：根据在另一个数组中给出的各自的权重计算数组中元素的加权平均值
np.mean()：数组中元素的算术平均值，可按轴计算
np.median()：中位数
np.amin()：计算数组中的元素沿指定轴的最小值
np.amax()：计算数组中的元素沿指定轴的最大值
np.ptp()：计算数组中元素最大值与最小值的差
```

## 日志检查

```
assert CONDITION, MESSAGE
```

## matplotlib绘图

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

## PIP

pip设置镜像源和代理
```
pip3 install torch -i https://mirrors.tencent.com/pypi/simple  --trusted-host mirrors.tencent.com --proxy proxy.xxx.com
pip3 install torch -i https://pypi.tuna.tsinghua.edu.cn/simple --trusted-host pypi.tuna.tsinghua.edu.cn
pip3 install torch -i https://mirrors.aliyun.com/pypi/simple   --trusted-host mirrors.aliyun.com
pip3 install torch -i https://pypi.mirrors.ustc.edu.cn/simple  --trusted-host pypi.mirrors.ustc.edu.cn
```

pip设置信任镜像源
```
# 命令行方式
pip config set global.index-url https://mirrors.tencent.com/pypi/simple/
pip config set global.extra-index-url https://mirrors.tencent.com/repository/pypi/tencent_pypi/simple
pip config set global.trusted-host mirrors.tencent.com

# 修改配置文件方式
$ cat ~/.config/pip/pip.conf
index-url = https://mirrors.tencent.com/pypi/simple/
extra-index-url = https://mirrors.tencent.com/repository/pypi/tencent_pypi/simple
trusted-host = mirrors.tencent.com
root-user-action = ignore
```

### Pypi国内镜像源
[腾讯](https://mirrors.tencent.com/pypi/simple)
[阿里云](https://mirrors.aliyun.com/pypi/simple)
[清华](https://pypi.tuna.tsinghua.edu.cn/simple)
[中科大](https://pypi.mirrors.ustc.edu.cn/simple)
[豆瓣](https://pypi.douban.com/simple)

## Conda

| command                                     | usage                                         |
| -----                                       | -----                                         |
| conda create --name py35 python=3.5         | create env                                    |
| conda remove --name py35 --all              | delete env                                    |
| conda create -n bak --clone src             | clone  env                                    |
| conda activate 3dlut                        | enter env                                     |
| conda deactivate                            | exit env                                      |
| conda info                                  | show conda info                               |
| conda info -e                               | show env                                      |
| conda install -n py35 numpy                 | install package                               |
| conda install --yes --file requirements.txt | install package via requirement file          |
| conda search numpy                          | search package                                |
| conda list -n py35                          | list package installed                        |
| conda update -n py35 numpy                  | update package                                |
| conda remove -n py35 numpy                  | delete package                                |
| conda update conda                          | update conda                                  |
| conda update anaconda                       | update anaconda                               |
| conda config --set auto_activate_base false | disable auto-activate base env while logining |
| conda clean -p                              | clean unused packages                         |
| conda clean -t                              | archive packages via tar                      |
| conda clean -y -all                         | remove all packages installed and cache       |
| conda env export > environment_droplet.yml  | export env to yml file                        |

```
conda config --show channels    # 显示当前镜像源配置
conda config --show-sources     # 显示镜像源配置文件路径
```

添加Conda镜像源：
```
conda config --add channels https://mirrors.tencent.com/anaconda/pkgs/r/
conda config --add channels https://mirrors.tencent.com/anaconda/pkgs/main/
conda config --add channels https://mirrors.tencent.com/anaconda/pkgs/free/
conda config --remove channels defaults
conda config --set show_channel_urls yes

conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/bioconda/
conda config --remove channels defaults
conda config --set show_channel_urls yes
```

修改conda的envs_dirs和pkgs_dirs：
可以通过编辑 \$HOME/.condarc 来实现。添加或修改 env_dirs 和 pkgs_dirs 配置项分别设置环境和缓存路径，按顺序第一个路径作为默认存储路径，搜索环境和缓存时按先后顺序在各目录中查找
```
envs_dirs:
  - /home/redleay/.conda/envs
  - /data/miniconda3/envs
pkgs_dirs:
  - /home/redleay/.conda/pkgs
  - /data/miniconda3/pkgs
```

conda安装应用报错：
```
Solving environment: failed with initial frozen solve. Retrying with flexible solve
```
解决办法：
```
$ conda -V # 查询conda版本
$ conda update -n base conda # 升级conda
conda update --all # 更新全部应用
```

## 其他

修改Linux系统语言编码，解决python获取Linux命令输出的中文字符乱码问题
```
export LC_ALL="en_US.utf8"
```
