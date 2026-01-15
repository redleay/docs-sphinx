# 流式编辑

## sed & ed

原地编辑
```
# 对于低版本sed，第1种in-place编辑方法在docker中存在权限问题，短期可使用其他方法暂时规避，长期可升级sed 4.8及以上版本解决
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

### MacOS执行sed报错invalid command code

原因是MacOS自带的sed等命令是基于BSD的，存在一些不足，对于`-i`选项和`\n`的处理不相同。

解决办法为改成`sed -i .bak`或者`sed -i ""`，或者安装gnu-sed:

```
brew install gnu-sed --with-default-names
```

## awk



## 其他

按行截取文本
```
head -n 100 FILE            # 截取前面100行
tail -n 100 FILE            # 截取后面100行
sed  -n '100,200p' FILE     # 截取100到200行
```

# 排序

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

先以"\_"为分隔符，第2列进行数值排序，后以"."为分隔符，第1列进行稳定的排序
```
sort -t . -k1,1 -s | sort -t \_ -k2n
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


# tr

删除字符串中的换行符
```
`tr -d '\n\r'`
```


# 归档压缩

## tar

创建归档，将bin和lib目录归档到压缩包
```
tar -c[zJ]vf archive.tar.gz bin/ lib/
```

解压归档，将压缩包解压到DIR目录
```
tar -x[zJ]vf archive.tar.gz -C DIR
```

查看归档，查看压缩包内文件列表
```
tar -t[zJ]vf archive.tar.gz
```

创建归档，将`tempdir/1.mp4`归档到压缩包，压缩包中去除`tempdir`前缀目录
```
tar -cvf output.tar -C tempdir 1.mp4
```

创建归档，将`tempdir/*.mp4`归档到压缩包，压缩包中去除`tempdir`前缀目录
```
tar -cvf output.tar -C tempdir $(find tempdir -maxdepth 1 -name "*.mp4" -exec basename {} \;)
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

## zip

创建归档
```
zip archive.zip -r ffmpeg/ pooltrans-1.0/
```

# ln链接

为file创建软链接softlink
```
ln -s file softlink
```

为file创建硬链接hardlink
```
ln file hardlink
```


# 进程管理

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

# tmux


列出所有快捷键，及其对应的tmux命令
```
tmux list-keys
```

列出所有tmux命令及其参数
```
tmux list-commands
```

列出当前所有tmux会话的信息
```
tmux info
```

重新加载当前的tmux配置
```
tmux source-file ~/.tmux.conf
```

创建会话
```
tmux    # 默认编号0
tmux new -s <name>
```

列出会话
```
tmux ls
tmux list-session
```

连接会话
```
tmux attach -t 0 # 使用会话编号
tmux attach -t <session-name> # 使用会话名称
```

解绑会话
```
tmux detach
```

切换会话
```
tmux switch -t 0 # 使用会话编号
tmux switch -t <session-name> # 使用会话名称
```

重命名会话
```
tmux rename-session -t 0 <new-name>
```

杀死会话
```
tmux kill-session -t 0 # 使用会话编号
tmux kill-session -t <session-name> # 使用会话名称
```

退出会话
```
exit
```

新建窗口
```
tmux new-window
tmux new-window -n <window-name> # 新建一个指定名称的窗口
```

切换窗口
```
tmux select-window -t <window-number>   # 切换到指定编号的窗口
tmux select-window -t <window-name>     # 切换到指定名称的窗口
```

重命名窗口
```
tmux rename-window <new-name>
```

其他快捷键及功能
```
Ctrl+b %：划分左右两个窗格。
Ctrl+b "：划分上下两个窗格。
Ctrl+b <arrow key>：切换到其他窗格
Ctrl+b ;：切换到上一个窗格。
Ctrl+b o：切换到下一个窗格。
Ctrl+b {：交换当前窗格与上一个窗格位置。
Ctrl+b }：交换当前窗格与下一个窗格位置。
Ctrl+b Ctrl+o：所有窗格向前移动一个位置，第一个窗格变成最后一个窗格。
Ctrl+b Alt+o：所有窗格向后移动一个位置，最后一个窗格变成第一个窗格。
Ctrl+b x：关闭当前窗格。
Ctrl+b !：将当前窗格拆分为一个独立窗口。
Ctrl+b z：当前窗格全屏显示，再使用一次会变回原来大小。
Ctrl+b Ctrl+<arrow key>：按箭头方向调整窗格大小。
Ctrl+b q：显示窗格编号。
Ctrl+b c：创建一个新窗口，状态栏会显示多个窗口的信息。
Ctrl+b p：切换到上一个窗口（按照状态栏上的顺序）。
Ctrl+b n：切换到下一个窗口。
Ctrl+b <number>：切换到指定编号的窗口，其中的<number>是状态栏上的窗口编号。
Ctrl+b w：从列表中选择窗口。
Ctrl+b ,：窗口重命名。
Ctrl+b [: 查看历史输出信息，按q退出
```

# 编译

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

# 打印

```
# 有风险，若`$INFO`中包含特殊字符`\`或`%`，则会引发转义，若特殊字符后未跟随合法的转义内容，则会转义失败
printf "$INFO"

# 安全，设置`%s`占位符，将`$INFO`解析为字符串，`printf`内部自动转义
printf "%s\n" "$INFO"
```

# 查看文件

```
xxd -l 500 test.yuv
od -d   test.yuv    # 1字节10进制显示
od -x   test.yuv    # 1字节16进制显示
od -td2 test.yuv    # 2字节10进制显示
od -tx2 test.yuv    # 2字节16进制显示
hexdump -b test.yuv # 1字节8进制显示
hexdump -d test.yuv # 2字节10进制显示
hexdump -x test.yuv # 2字节16进制显示
hexdump -C -s 2000 -n 1000 test.mp4 # 从2000 Byte开始，打印1000 Byte，同时以16进制和ASCII打印
```

# 账号

查看账号user的信息
```
id user
```

修改账号user和group的id
```
usermod -u id user
groupmod -g id group
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

# 网络

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

# 系统管理

```
uname -a
cat /etc/os-release
cat /etc/redhat-release
cat /etc/centos-release
hostnamectl
```

# yum软件管理

```
yum -y install https://packages.endpointdev.com/rhel/7/os/x86_64/endpoint-repo.x86_64.rpm
```

# 查看系统报错

```
dmesg -T
```

将`dmesg`的输出时间转换为人类可读时间
```
timestamp=105435829.344746
unix_time=`echo "$(date +%s) - $(cat /proc/uptime | cut -f 1 -d' ') + ${timestamp}" | bc`
stamp=`echo "scale=0; ${unix_time} / 1" | bc`
echo ${stamp}
date -d @${stamp} "+%Y-%m-%d %H:SM:%S"
```

# 中文相关

下载文件时中文文件名不转义
```
wget --restrict-file-names=nocontrol $URL
```

设置terminal终端支持中文显示，避免中文乱码
```
LANG="zh_CN.UTF-8"
```

列出所有支持中文的字体
```
fc-list :lang=zh
```

安装中文字体
```
# yum安装
sudo yum install wqy-zenhei-fonts wqy-microhei-fonts google-noto-cjk-fonts

# 手动安装
sudo cp sudo cp xxx-Regular.ttf /usr/share/fonts/
cd /usr/share/fonts
sudo mkfontscale
sudo fc-cache -fv
```

# 其他常用命令

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
find . -type f -print0 | xargs -0 ls -ld
find . | tr " " "\?" | xargs ls -ld
```

valgrind检测内存泄漏
```
valgrind --tool=memcheck --leak-check=full --show-leak-kinds=all ./x265
```


运行`sudo yum install graphviz ghostscript`报错：
```
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


# Shell脚本执行选项

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

# 环境

设置动态库搜索路径
```
export LD_LIBRARY_PATH=/path/to/lib:$LD_LIBRARY_PATH
```

查看库文件中的string，查看glic版本
```
strings /lib64/libc.so.6 \| grep GLIBC
```

# 变量

## 基础操作

变量引用
```
$var
${var}
```

引用变量时带默认值
```
${var-default}  # 如果var没有定义，则返回default，否则返回var的值
${var:-default} # 如果var没有定义或为空，则返回default，否则返回var的值
```

## 环境变量

```
export back_dir=/home/backup
env | grep "backdir"
```

## 位置变量

```
$1 $2 ${10}
```

## 预定义变量

```
$0 脚本名称 默认输出路径加脚本名，可以使用basename来只输出脚本的名称不包含路径，dirname可以输出脚本所在的文件夹名称
$* 所有的参数，被作为一个单词，注意 $* 必须被""引用
$@ 所有的参数，每个参数都是一个独立的""引用字串，这就意味着参数被完整地传递
$# 参数的个数
$$ 当前进程的PID
$! 上一个后台进程的PID
$? 上一个命令的返回值，0表示成功
```

# 字符串

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

# 字符串判断

| 命令 | 作用 |
| --- | --- |
| [ -z STRING ] | 如果STRING的长度为零则为真，即判断是否为空，空即是真 |
| [ -n STRING ] | 如果STRING的长度非零则为真，即判断是否为非空，非空即是真 |
| [ STRING ]    | 如果STRING不为空则为真，与-n类似 |
| [ STRING1 = STRING2 ] | 如果两个字符串相同则为真 |
| [ STRING1 == STRING2 ] | 如果字符串相同则为真, “=” instead of “==” for strict POSIX compliance |
| [ STRING1 != STRING2 ] | 如果字符串不相同则为真 |
| [ STRING1 < STRING2 ] | 如果 “STRING1” sorts before “STRING2”则为真, lexicographically in the current locale |
| [ STRING1 > STRING2 ] | 如果 “STRING1” sorts after “STRING2”则为真, lexicographically in the current locale |

# 数值判断

| 命令 | 作用 |
| --- | --- |
| INT1 -eq INT2 | `=`:  INT1和INT2两数相等为真 |
| INT1 -ne INT2 | `<>`: INT1和INT2两数不等为真 |
| INT1 -gt INT2 | `>`:  INT1大于INT1为真 |
| INT1 -ge INT2 | `>=`: INT1大于等于INT2为真 |
| INT1 -lt INT2 | `<`:  INT1小于INT2为真 |
| INT1 -le INT2 | `<=`: INT1小于等于INT2为真 |


# 文件/目录判断

| 命令 | 作用 |
| --- | --- |
| [ -d DIR  ] | 如果 FILE 存在且是一个目录则为真 |
| [ -e FILE ] | 如果 FILE 存在则为真 |
| [ -a FILE ] | 如果 FILE 存在则为真 |
| [ -s FILE ] | 如果 FILE 存在且大小不为0则为真 |
| [ -r FILE ] | 如果 FILE 存在且是可读的则为真 |
| [ -w FILE ] | 如果 FILE 存在且是可写的则为真 |
| [ -x FILE ] | 如果 FILE 存在且是可执行的则为真 |
| [ -N FILE ] | 如果 FILE 存在且在上次读取之后被修改了则为真 |
| [ -b FILE ] | 如果 FILE 存在且是一个块特殊文件则为真 |
| [ -c FILE ] | 如果 FILE 存在且是一个字特殊文件则为真 |
| [ -f FILE ] | 如果 FILE 存在且是一个普通文件则为真 |
| [ -p FILE ] | 如果 FILE 存在且是一个名字管道(FifO)则为真 |
| [ -L FILE ] | 如果 FILE 存在且是一个符号连接则为真 |
| [ -S FILE ] | 如果 FILE 存在且是一个套接字则为真 |
| [ -g FILE ] | 如果 FILE 存在且设置了SGID则为真 |
| [ -u FILE ] | 如果 FILE 存在且设置了SUID则为真 |
| [ -k FILE ] | 如果 FILE 存在且设置了粘制位则为真 |
| [ -O FILE ] | 如果 FILE 存在且属有效用户ID则为真 |
| [ -G FILE ] | 如果 FILE 存在且属有效用户组则为真 |
| [ FILE1 -nt FILE2 ] | 如果 FILE1 修改时间比 FILE2 更晚，或者 FILE1 存在且 FILE2 不存在则为真 |
| [ FILE1 -ot FILE2 ] | 如果 FILE1 修改时间比 FILE2 要早，即 FILE1 更老，或者 FILE2 存在且 FILE1 不存在则为真 |
| [ FILE1 -ef FILE2 ] | 如果 FILE1 和 FILE2 指向相同的设备和节点号则为真 |
| [ -t FD ]   | 如果文件描述符 FD 打开且指向一个终端则为真 |

具体可以参考`man test`

# 语法

循环
```
for i in {1..100}; do echo $i; done
for i in `seq 1 2 100`; do echo $i; done    # 等差数列，step为2
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

# 函数

```
funWithParam(){
    echo "$1!"
    echo "$2!"
    return $1
}

funWithParam 1 2
```

## 位置变量
··
