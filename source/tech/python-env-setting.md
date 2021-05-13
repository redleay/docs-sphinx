# Python相关环境配置

## PIP

pip设置镜像源和代理
```
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

可以通过编辑 \$HOME/.condarc 来实现。添加或修改 \$HOME/.condarc 中的 env_dirs 和 pkgs_dirs 配置项分别设置环境和缓存路径，按顺序第一个路径作为默认存储路径，搜索环境和缓存时按先后顺序在各目录中查找
```
envs_dirs:
  - /home/redleay/.conda/envs
  - /data/miniconda3/envs
pkgs_dirs:
  - /home/redleay/.conda/pkgs
  - /data/miniconda3/pkgs
```

## 其他

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
