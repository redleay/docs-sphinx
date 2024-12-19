# LinuxShell语法和使用

## Shell脚本执行选项

读取shell脚本，但不实际执行
```
sh -n run.sh
```

进入跟踪方式，显示所执行的每一条命令
```
sh -x run.sh
```

从string中读取命令
```
sh -c "string"
```

## 环境

设置动态库搜索路径
```
export LD_LIBRARY_PATH=/path/to/lib:$LD_LIBRARY_PATH
```

查看库文件中的string，查看glic版本
```
strings /lib64/libc.so.6 \| grep GLIBC
```


## 账号管理

查看账号user的信息
```
id user
```

修改HOME目录：
```
usermod -d NEWHOME -u uid USERNAME
```

将user添加到group中
```
sudo usermod -a -G group user
sudo usermod -a -G group1,group2 user
```

更改user的primary group
```
sudo usermod -g group user
```

sudo groupadd groupname
sudo groupdel groupname

## 字符串操作

获取string的长度
```
${#string}
```

在string中, 从位置position开始提取子串

```
${string:position}
```

在string中, 从位置position开始提取长度为length的子串
```
${string:position:length}
```

从变量string的开头, 删除最短匹配substring的子串
```
${string#substring}
```

从变量string的开头, 删除最长匹配substring的子串
```
${string##substring}
```

从变量string的结尾, 删除最短匹配substring的子串
```
${string%substring}
```

从变量string的结尾, 删除最长匹配substring的子串
```
${string%%substring}
```

删除变量string结尾的空格
```
${string%%[[:space:]]\*}
```

使用replacement, 来代替第一个匹配的substring
```
${string/substring/replacement}
```

使用replacement, 代替所有匹配的substring
```
${string//substring/replacement}
```

如果string的前缀匹配substring, 那么就用replacement来代替匹配到的substring
```
${string/#substring/replacement}
```

如果string的后缀匹配substring, 那么就用replacement来代替匹配到的substring
```
${string/%substring/replacement}
```


## sed & ed

原地编辑
```
# 第1种in-place编辑方法在docker中存在权限问题，推荐使用其他方法
sed -i 'xxx' FILE
sed -ci 'xxx' FILE
sed 'xxx' FILE > NEWFILE
printf "%s\n" '1,$s/search/replace/g' wq | ed -s file
```

删除第N行
```
sed 'Nd' FILE
```

删除最后一行
```
sed '$d' FILE
```

删除第5到7行
```
sed '5,7d' FILE
```

删除第5、7行
```
sed '5d;7d' FILE
```

只保留第5到7行
```
sed '5,7!d' FILE
```

对第3行，将其中的aaa替换为fff
```
sed '3s/aaa/fff/' FILE
```

对所有行，在行首插入fff
```
sed 's/^/fff/' FILE
```

找出包含xxx的行，并将其中的aaa替换为fff
```
sed '/xxx/s/aaa/fff/g' FILE
```

对第1行，将其中的#号或是\*号替换为fff
```
sed '1s/[#\*]/fff/gp' FILE
```

删除空行
```
sed '/^[ ]\*$/d' FILE
```

删除包含error的行
```
sed '/error/d' FILE
```

删除不包含error的行
```
sed '/error/!d' FILE
```

查找inputXX并替换为变量$v的内容，使用双引号
```
sed "s#inputXX#$v#" FILE
```

查找数字
```
sed "s/[0-9]//" FILE
```

匹配分组
```
sed "s/\(XXX\)/\1,\1/" FILE
```

第3行前插入New Line
```
sed '3i New line' FILE
```

最后1行后插入New Line
```
sed '$a New line' FILE
```

## 排序

以数值来排序
```
sort -n
```

以"\_"为分隔符，以第2列进行数值排序
```
sort -t \_ -k 2 -n
```

以"\_"为分隔符，先第3列进行稳定的数值排序，后第2列进行数值排序
```
sort -t \_ -k2n -k3n -s
```

sort -t . -k1,1 -s|  先以"\_"为分隔符，第2列进行数值排序，后以"."为分隔符，第1列进行稳定的排序
```
sort -t \_ -k2n
```

排序并删除重复行
```
sort -u
sort | uniq
```

随机排序
```
shuf
```


## tr

删除字符串中的换行符
```
`tr -d '\n\r'`
```


## 归档压缩

创建归档，将bin和lib目录归档到压缩包
```
tar -c[zj]vf archive.tar.gz bin/ lib/
```

解压归档，将压缩包解压到DIR目录
```
tar -x[zj]vf archive.tar.gz -C DIR
```

查看归档，查看压缩包内文件列表
```
tar -t[zj]vf archive.tar.gz
```

创建归档
```
zip archive.zip -r ffmpeg/ pooltrans-1.0/
```

创建归档，并分割为每个4000M
```
tar -cvf All.tar All/ | split -b 4000M -d -a 1 - Split.tar.
```

解压分割后的压缩文件
```
cat Split.tar.\* | tar -xv
```

从压缩包中获取所有文件列表，排除目录
```
tar -tzvf archive.tar.gz | awk '$1 ~ /^-/{print $NF}'
```


## ln链接

为file创建软链接softlink
```
ln -s file softlink
```

为file创建硬链接hardlink
```
ln file hardlink
```


## 进程管理

查看进程stdout
```
strace -ewrite -p $PID
```

查看正在后台运行的任务状态
```
jobs
```

将任务调度到前台运行，不带jobnumber时表示对最后一个进程操作
```
fg jobnumber
```

将任务调度到后台运行，不带jobnumber时表示对最后一个进程操作
```
bg jobnumber
```

将前台任务调度到后台中暂停，显示jobnumber
```
Ctrl-Z
```


## 编译

打印make执行的命令
```
make VERBOSE=1
```

指定make install的安装路径
```
make install PREFIX=/usr/local/test
```

查询exe或lib是否存在\_debug开头的段，有则携带debug信息
```
readelf -S exe\|lib
```


## 语法

循环
```
for i in {1..100}; do echo $i; done
```

循环，循环起止由变量$m和$n控制
```
for i in ((i=0; i<8; i++)); do echo $i; done
```

循环，循环起止由变量$m和$n控制
```
for i in $(seq $m $n); do echo $i; done
```

循环文件名带空格的目录
```
for f in \`ls . | tr " " "\?"\`; do echo $f; done
```

数组定义
```
bitrate=(4000 3200 2800 2400)
```

引用和展开数组
```
for r in ${bitrate[@]}
```

引用数组元素
```
i=2; echo ${bitrate[i]}
```


变量乘法，返回900.0
```
r=750; f=1.2; echo "scale=0;$r\*$f" | bc
```

浮点数取整
```
r=900.0; r=$(echo $r | awk '{print int($r)}')
```

判断是否相等
```
if [ $i -eq 0 ]
```

判断目录是否存在
```
if [ -d "./out" ]
```

数字变量格式化为字符串
```
fmt=`printf "%05d" $i`
```

变量自加1
```
((i++))
```

引用循环变量
```
while read line; do cp "$line" $dst; done < $file
```

设置IFS分隔符
```
IFS=$'\n'
```

并行处理
```
proc=10     # parallel number
mkfifo ./fifo && exec 3<> ./fifo && rm -rf ./fifo; for ((i=0; i<$proc; i++)) do echo >&3; done
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

## 网络

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

实时网速和流量情况
```
sudo nload eth0 -m
sudo nload -m
vnstat -i eth0 -l  # 查看实时流量
iftop -i eth1
nethogs -d 5 eth0  # 每5秒种刷新1次
```


## yum软件包管理


```
yum -y install https://packages.endpointdev.com/rhel/7/os/x86_64/endpoint-repo.x86_64.rpm
```


## 其他常用命令

下载文件时中文文件名不转义
```
wget --restrict-file-names=nocontrol $URL
```

设置terminal终端支持中文显示，避免中文乱码
```
LANG="zh_CN.UTF-8"
```

统计运行时间
```
(time -f "pencent %P real %e cpu %S %U men %M %K" ffmpeg) > log.txt 2>&1
```

查找文件并计算md5
```
find . -type f -print0 | xargs -0 md5sum > checksum.md5
find . -not -path '*/\.*' -not -name '*.log' -type f -exec md5sum {} + > checksum.md5   # 排除隐藏目录、隐藏文件和.log文件
```

列出当前目录下的所有目录名
```
ls -d */
ls -l | grep "^d" | awk '{print $9}'
```

列出目录和文件的完整路径
```
find . -type f -print0 | xargs -0 ls -d
find . | tr " " "\?" | xargs ls -ld
```

valgrind检测内存泄漏
```
valgrind --tool=memcheck --leak-check=full --show-leak-kinds=all ./x265
```


运行`sudo yum install graphviz ghostscript`报错：
```
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors-tlinux.tencentyun.com
 * epel: mirrors.tencentyun.com
 * extras: mirrors-tlinux.tencentyun.com
 * tlinux: mirrors-tlinux.tencentyun.com
 * updates: mirrors-tlinux.tencentyun.com
base                                                                                                        | 3.6 kB  00:00:00     
centos-sclo-rh                                                                                              | 3.0 kB  00:00:00     
centos-sclo-sclo                                                                                            | 3.0 kB  00:00:00     
https://packages.endpointdev.com/rhel/2.2/os/x86_64/repodata/repomd.xml: [Errno 14] HTTPS Error 404 - Not Found
Trying other mirror.
To address this issue please refer to the below wiki article 

https://wiki.centos.org/yum-errors

If above article doesn't help to resolve this issue please use https://bugs.centos.org/.

 One of the configured repositories failed (End Point repository),
 and yum doesn't have enough cached data to continue. At this point the only
 safe thing yum can do is fail. There are a few ways to work "fix" this:

     1. Contact the upstream for the repository and get them to fix the problem.

     2. Reconfigure the baseurl/etc. for the repository, to point to a working
        upstream. This is most often useful if you are using a newer
        distribution release than is supported by the repository (and the
        packages for the previous distribution release still work).

     3. Run the command with the repository temporarily disabled
            yum --disablerepo=endpoint ...

     4. Disable the repository permanently, so yum won't use it by default. Yum
        will then just ignore the repository until you permanently enable it
        again or use --enablerepo for temporary usage:

            yum-config-manager --disable endpoint
        or
            subscription-manager repos --disable=endpoint

     5. Configure the failing repository to be skipped, if it is unavailable.
        Note that yum will try to contact the repo. when it runs most commands,
        so will have to try and fail each time (and thus. yum will be be much
        slower). If it is a very temporary problem though, this is often a nice
        compromise:

            yum-config-manager --save --setopt=endpoint.skip_if_unavailable=true

failure: repodata/repomd.xml from endpoint: [Errno 256] No more mirrors to try.
https://packages.endpointdev.com/rhel/2.2/os/x86_64/repodata/repomd.xml: [Errno 14] HTTPS Error 404 - Not Found
```
使用以下命令`sudo yum --disablerepo=endpoint install graphviz ghostscript`解决


