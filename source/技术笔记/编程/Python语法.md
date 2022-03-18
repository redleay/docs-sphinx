# Python语法

## 数据类型处理
List转Set: `set(listA)`

List转Dict: `dict.fromkeys(listA)`

List去重: `list(set(listA)`

List追加合并: `listC = listA + listB`

List连接: `chain(bars_576p, bars_720p, bars_1080p)`

## Numpy
List转numpy.array：`np.array(listA)`
numpy.array转List: `listA = numpyarrayA.tolist()`

numpy内置索引: `rgb[rgb < 0] = 0`

numpy.array：可直接通过`+-*/`实现逐个元素的加减乘除运算

## 迭代式
单List循环: `[v for v in values]`

双List循环: `[w for v in values for w in v.get("width")]`

单List循环返回索引值: `[i for i,v in enumerate(vmaf_list_wesee) if v < 82.0]`

多List循环返回特定值: `[b for w,b in zip(width_list, bitrate_list) if w == 720]`


## 数学计算

```
math.floor()
math.ceil ()
np.arange(low, high, step)
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
pip3 install tensorflow==1.10.0 -i https://mirrors.tencent.com/pypi/simple --trusted-host mirrors.tencent.com
pip3 install tensorflow==1.10.0 -i https://pypi.tuna.tsinghua.edu.cn/simple --trusted-host pypi.tuna.tsinghua.edu.cn
pip3 install tensorflow==1.10.0 -i https://pypi.tuna.tsinghua.edu.cn/simple --proxy proxy.xxx.com
```
### Pypi国内镜像源

[腾讯](https://mirrors.tencent.com/pypi/simple)
[阿里云](http://mirrors.aliyun.com/pypi/simple)
[豆瓣](http://pypi.douban.com/simple)
[清华](https://pypi.tuna.tsinghua.edu.cn/simple)
[中科大](https://pypi.mirrors.ustc.edu.cn/simple)

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

## TensorBoard

执行`tensorboard --logdir=LOG_PATH`，然后在浏览器中访问`http://localhost:6006/`便可以查看到图形

## 其他

修改Linux系统语言编码，解决python获取Linux命令输出的中文字符乱码问题
```
export LC_ALL="en_US.utf8"
```

查看编译Pytorch release版本时使用的CUDA版本
```
>>> import torch
>>> torch.version.cuda
```

查看Pytorch实际运行时使用的CUDA版本
```
>>> import torch
>>> import torch.utils
>>> import torch.utils.cpp_extension
>>> torch.utils.cpp_extension.CUDA_HOME
```

使用torch.jit.script()将pth类型模型导出为pt类型时
```
serialized_model = torch.jit.script(netG)
serialized_model.save('model.pt')
```
输出以下报错：
```
torch.jit.frontend.NotSupportedError: Compiled functions can't take variable number of arguments or use keyword-only arguments with defaults:
```
原因：jit不支持DataParallel，解决方法为不使用nn.DataParallel()，且将模型参数的key的module.前缀删除，可参考[这篇文章](https://szukevin.site/2021/02/27/MODNet%E8%BD%AC%E6%88%90torchscript%E5%BD%A2%E5%BC%8F%E9%81%87%E5%88%B0%E7%9A%84%E5%9D%91/)
