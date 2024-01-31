# LinuxShell语法

## Shell脚本执行选项
| 示例           | 功能                                 |
| ------------   | ------------                         |
| sh -n run.sh   | 只读取shell脚本，但不实际执行        |
| sh -x run.sh   | 进入跟踪方式，显示所执行的每一条命令 |
| sh -c "string" | 从strings中读取命令                  |

## 字符串操作
| 示例                             | 功能                                                                          |
| ------------                     | ------------                                                                  |
| ${#string}                       | 获取$string的长度                                                             |
| ${string:position}               | 在$string中, 从位置$position开始提取子串                                      |
| ${string:position:length}        | 在$string中, 从位置$position开始提取长度为$length的子串                       |
| ${string#substring}              | 从变量$string的开头, 删除最短匹配$substring的子串                             |
| ${string##substring}             | 从变量$string的开头, 删除最长匹配$substring的子串                             |
| ${string%substring}              | 从变量$string的结尾, 删除最短匹配$substring的子串                             |
| ${string%%substring}             | 从变量$string的结尾, 删除最长匹配$substring的子串                             |
| ${string%%[[:space:]]\*}         | 删除变量$string结尾的空格                                                     |
| ${string/substring/replacement}  | 使用$replacement, 来代替第一个匹配的$substring                                |
| ${string//substring/replacement} | 使用$replacement, 代替所有匹配的$substring                                    |
| ${string/#substring/replacement} | 如果$string的前缀匹配$substring, 那么就用$replacement来代替匹配到的$substring |
| ${string/%substring/replacement} | 如果$string的后缀匹配$substring, 那么就用$replacement来代替匹配到的$substring |


## sed
| 示例                           | 功能                                    |
| ------------                   | ------------                            |
| sed 'Nd'    file               | 删除第N行                               |
| sed '$d'    file               | 删除最后一行                            |
| sed '5,7d'  file               | 删除第5到7行                            |
| sed '5d;7d' file               | 删除第5、7行                            |
| sed '5,7!d' file               | 只保留第5到7行                          |
| sed -i '3s/aaa/fff/' file      | 对第3行，将其中的aaa替换为fff           |
| sed -i 's/^/fff/' file         | 对所有行，在行首插入fff                 |
| sed -i '/xxx/s/aaa/fff/g' file | 找出包含xxx的行，并将其中的aaa替换为fff |
| sed -i '1s/[#\*]/fff/gp' file  | 对第1行，将其中的#号或是\*号替换为fff   |
| sed -i '/^[ ]\*$/d' file       | 删除空行                                |
| sed -i "s#inputXX#$v#" file    | 查找inputXX并替换为变量$v的内容，使用双引号 |
| sed -i "s/[0-9]//" file        | 查找数字                                |
| sed -i "s/\(XXX\)/\1,\1/" file | 匹配分组                                |


## 排序
| 示例                                | 功能                                                                          |
| ------------                        | ------------                                                                  |
| sort -n                             | 以数值来排序                                                                  |
| sort -t \_ -k 2 -n                  | 以"\_"为分隔符，以第2列进行数值排序                                           |
| sort -t \_ -k2n -k3n -s             | 以"\_"为分隔符，先第2列进行数值排序，后第3列进行稳定的数值排序                |
| sort -t \_ -k2n | sort -t . -k1,1 -s|  先以"\_"为分隔符，第2列进行数值排序，后以"."为分隔符，第1列进行稳定的排序    |
| shuf                                | 随机排序                                                                      |

## tr
tr -d '\n\r' 删除字符串中的换行符

## 压缩
| 示例                                                         | 功能                                 |
| ------------                                                 | ------------                         |
| tar -c[zj]vf archive.tar.gz bin/ lib/                        | 创建归档，将bin和lib目录归档到压缩包 |
| tar -x[zj]vf archive.tar.gz -C DIR                           | 解压归档，将压缩包解压到DIR目录      |
| tar -t[zj]vf archive.tar.gz                                  | 查看归档，查看压缩包内文件列表       |
| zip archive.zip -r ffmpeg/ pooltrans-1.0/                    | 创建归档                             |
| tar -cvf All.tar All/ \| split -b 4000M -d -a 1 - Split.tar. | 创建归档，并分割为每个4000M          |
| cat Split.tar.\* \| tar -xv                                  | 解压分割后的压缩文件                 |

## ln链接
| 示例                | 功能                     |
| ------------        | ------------             |
| ln -s file softlink | 为file创建软链接softlink |
| ln file hardlink    | 为file创建硬链接hardlink |

## 前后台调度
| 示例         | 功能                                                        |
| ------------ | ------------                                                |
| jobs         | 查看正在运行的任务状态                                      |
| fg jobnumber | 将任务调度到前台运行，不带jobnumber时表示对最后一个进程操作 |
| bg jobnumber | 将任务调度到后台运行，不带jobnumber时表示对最后一个进程操作 |
| Ctrl-Z       | 将前台任务调度到后台中暂停，显示jobnumber                   |

## 环境
| 示例                                                    | 功能               |
| ------------                                            | ------------       |
| export LD\_LIBRARY\_PATH=$LD\_LIBRARY_PATH:/path/to/lib | 设置动态库搜索路径 |
| strings /lib64/libc.so.6 \| grep GLIBC                  | 查看库文件中的string |

## 编译
| 示例                | 功能               |
| ------------        | ------------       |
| make VERBOSE=1      | 打印make执行的命令 |
| make install PREFIX=/usr/local/test      | 指定make install的安装路径 |
| readelf -S exe\|lib | 查询exe或lib是否存在\_debug开头的段，有则携带debug信息 |

## 语法
| 示例                                    | 功能                           |
| ------------                            | ------------                   |
| for i in {1..100}; do echo $i; done     | 循环                           |
| for i in ((i=0; i<8; i++)); do echo $i; done | 循环，循环起止由变量$m和$n控制 |
| for i in $(seq $m $n); do echo $i; done | 循环，循环起止由变量$m和$n控制 |
| for f in `ls . | tr " " "\?"`; do echo $f; done | 循环文件名带空格的目录 |
| bitrate=(4000 3200 2800 2400)           | 数组定义                       |
| for r in ${bitrate[@]}                  | 引用和展开数组                 |
| i=2; echo ${bitrate[i]}                 | 引用数组元素                   |
| r=750; f=1.2; echo "scale=0;$r\*$f" \| bc | 变量乘法，返回900.0 |
| r=900.0; r=$(echo $r \| awk '{print int($r)}')  | 浮点数取整 |
| if [ $i -eq 0 ] | 判断是否相等 |
| if [ -d "./out" ] | 判断目录是否存在 |
| fmt=`printf "%05d" $i` | 数字变量格式化为字符串 |
| ((i++)) | 变量自加1 |
| while read line; do cp "$line" $dst; done < $file |
| IFS=$'\n' |


## 其他常用命令

修改HOME目录：
```
usermod -d NEWHOME -u uid USERNAME
```

排序并删除重复行
```
sort | uniq
```

下载文件时中文文件名不转义
```
wget --restrict-file-names=nocontrol $URL
```

设置terminal终端支持中文显示
```
LANG="zh_CN.UTF-8"
```

```
(time -f "pencent %P real %e cpu %S %U men %M %K" ffmpeg) > log.txt 2>&1
```

列出目录和文件的完整路径
```
find . | tr " " "\?" | xargs ls -ld
find . -type f -print0 | xargs -0 ls -ld
```

设置Shell代理
```
export http_proxy="http://proxy.xxx.com"
export https_proxy="http://proxy.xxx.com"
export no_proxy="mirrors.xxx.com
```

取消Shell代理
```
unset http_proxy
unset https_proxy
unset no_proxy
```

并行处理
```
proc=10     # parallel number
mkfifo ./fifo && exec 3<> ./fifo && rm -rf ./fifo
for ((i=0; i<$proc; i++))
do
  echo >&3
done

for f in `ls *.dng`
do
  read -u 3

  {
    # do things
    echo >&3
  }&
done

wait
```

实时网速和流量情况
```
sudo nload eth0 -m
sudo nload -m
vnstat -i eth0 -l  # 查看实时流量
iftop -i eth1
nethogs -d 5 eth0  # 每5秒种刷新1次
```

升级gcc版本到7.x.x
```
sudo yum install centos-release-scl
sudo yum install devtoolset-7-gcc devtoolset-7-gcc-c++  # devtoolset-7-gcc*
scl enable devtoolset-7 bash        # shall execute for every terminai
source /opt/rh/devtoolset-7/enable
```

查看进程stdout
```
strace -ewrite -p $PID
```

valgrind检测内存泄漏
```
valgrind --tool=memcheck --leak-check=full ./x265
```
