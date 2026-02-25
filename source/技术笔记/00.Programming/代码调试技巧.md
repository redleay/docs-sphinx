[toc]

# 通用技巧

查看已启动进程的stdout和stderr
```
strace -ewrite -p $PID 2>&1
```

重定向已启动进程的stdout和stderr

```
gdb -p $PID
(gdb) p dup2(open("/dev/null", 0), 1)  # 1表示stdout，具体路径为/proc/$PID/fd/1
(gdb) p dup2(open("/dev/null", 0), 2)  # 2表示stderr，具体路径为/proc/$PID/fd/2
(gdb) detach
(gdb) quit
```

# C代码调试

## 内存检测

```
valgrind --tool=memcheck --leak-check=full --show-leak-kinds=all $CMD
```

## 性能分析

### gprof

使用方法为：
- 采用静态编译和链接，编译时打开`-pq`选项
- 正常执行$CMD
- 分析结果gprof $CMD > prof.txt

### Valgrind

先执行
```
valgrind --tool=callgrind --separate-threads=yes $CMD
```

然后使用QCacheGrind可视化结果

### gperftools

#### 安装gperftools

先安装依赖库，然后编译安装gperftools，会安装`libprofiler.so`、 `libtcmalloc.so`等库和性能报告生成工具`pprof`
```
./configure && make && make install
sudo yum install -y libunwind
```

#### 启用profiler库

方法一

编译链接时加上`-lprofiler`选项

方法二

使用`LD_PRELOAD`环境变量将`libprofiler.so` hook进程序，这种方法不需要重新编译程序

```
LD_PRELOAD=/usr/local/lib/libprofiler.so $CMD
```

#### 收集性能数据

```
CPUPROFILE=server.prof CPUPROFILE_FREQUENCY=500 ./server
```

server.prof为输出结果，500为每秒收集频率

#### 导出报告

```
sudo apt install graphviz ghostscript
pprof --pdf ./server server.prof > perf.pdf
```

## GDB断点调试

启动gdb
```
# 方法1
gdb --args ./ffmpeg -i input.mp4 output.mp4

# 方法2
gdb ./ffmpeg
set args -i input.mp4 output.mp4
```

### 常用命令

```
attach PID                          # 附上PID进程并进入调试
show dir                            # 查看源码搜索路径
dir srcpath1:srcpath2               # 添加源码搜索路径，用冒号分开
set substitute-path srcpath dstpath # 映射源码搜索路径，srcpath字符串替换为为dstpath
tb                                  # 设置临时断点
watch                               # 设置监控断点，当监控变量或内存的内容变化时停止运行
display                             # 自动打印调试信息
undisplay                           # 取消自动打印调试信息
x/nfu addr                          # 打印内存块，n: 单元数量，f: 输出格式(bodx, 2-8-10-16进制)，u: 单元长度(bhwg, 1-2-4-8字节)
set scheduler-locking off           # 执行所有thread
set scheduler-locking on            # 只执行当前thread
set scheduler-locking step          # 单步时，除了next过一个函数的情况以外，只执行当前thread
```

### FAQ

Q: gdb调试时输出信息`Program received signal SIGTRAP, Trace/breakpoint trap`  
A: 原因是编译和调试的动态库版本不匹配，如编译时使用了devtoolset-7中的gcc-7.3.1，但调试时LD_LIBRARY_PATH没有添加devtoolset-7动态库路径，gdb仍然使用系统默认的gcc-4.8.5动态库，在LD_LIBRARY_PATH添加devtoolset-7动态库路径即可解决


## core文件分析

方法1
```
gdb ./ffmpeg
core-file core.xxx
bt
```

方法2
```
gdb -c core.xxx
file ./ffmpeg
bt
```

方法3
```
gdb ./ffmpeg core.xxx
bt
```

# Python代码调试

## PDB

启动方法，非侵入式方法（不修改源代码，在命令行下直接运行）
```
python3 -m pdb filename.py
```

启动方法，侵入式方法（需要在被调试代码中添加一行代码，然后正常运行代码）
```
import pdb;pdb.set_trace()
```

### 常用命令
```
l                           # 查看代码 查看当前位置前后11行代码，符号-->标示当前位置
ll                          # 查看代码 查看当前函数或框架的所有源代码
b/tbreak                    # 查看断点 查看断点/临时断点
b/tbreak filename:lineno    # 设置断点 设置断点/临时断点
b/tbreak functionname       # 设置断点 设置断点/临时断点
b 1131, i==30 and j==74     # 设置断点 设置条件断点
cl                          # 清除断点 清除所有断点（包括临时断点）
cl filename:lineno          # 清除断点 清除指定断点
cl bpnumber [bpnumber ...]  # 清除断点 清除指定断点
n                           # 调试 执行下一行（不进入函数体）
s                           # 调试 执行下一行（能进入函数体）
r                           # 调试 执行下一行（在函数中时会执行到函数返回处）
c                           # 调试 持续执行，直到遇到断点
unt lineno                  # 调试 持续执行，直到指定行（或断点）
j lineno                    # 调试 直接跳转到指定行（被跳过的代码不执行）
p expression                # 查看变量值 打印变量值
p obj.__dict__              # 查看变量值 查看类对象的所有成员变量值
a                           # 查看函数参数 在函数中时打印函数的参数和参数的值
whatis expression           # 打印变量类型 打印表达式的类型，常用来打印变量值
w                           # 打印堆栈信息 最新的帧在最底部，箭头表示当前帧
interact                    # 启动一个python交互式解释器 使用当前代码的全局命名空间（使用ctrl+d返回pdb）
q                           # 退出 退出pdb
```
