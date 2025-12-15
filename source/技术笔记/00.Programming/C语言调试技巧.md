# C语言调试技巧

## 查看已启动进程的stdout和stderr
```
strace -ewrite -p $PID 2>&1
```

## 内存检测

```
valgrind --tool=memcheck --leak-check=full --show-leak-kinds=all $CMD
```

## 性能瓶颈分析

### gprof

采用静态编译和链接，编译时打开`-pq`选项
正常执行$CMD
分析结果gprof $CMD > prof.txt

### Valgrind

执行

```
valgrind --tool=callgrind --separate-threads=yes $CMD
```

使用QCacheGrind可视化结果

### gperftools

安装依赖库，下载和编译安装gperftools，会安装libprofiler.so、libtcmalloc.so等库文件, 性能报告生成工具pprof

```
./configure && make && make install
sudo yum install -y libunwind
```

启用profiler库，以下两种方法二选一

1. 编译链接时加上`-lprofiler`选项
2. 使用LD_PRELOAD环境变量将libprofiler.so hook进程序，这种方法不需要重新编译程序：`LD_PRELOAD=/usr/local/lib/libprofiler.so $CMD

收集性能数据，server.prof为输出结果，500为每秒收集频率

```
CPUPROFILE=server.prof CPUPROFILE_FREQUENCY=500 ./server
```

导出报告

```
sudo apt install graphviz ghostscript
pprof --pdf ./server server.prof > perf.pdf
```

## GDB

启动gdb
```
# 方法1
gdb --args ./ffmpeg -i input.mp4 output.mp4

# 方法2
gdb ./ffmpeg
set args -i input.mp4 output.mp4
```

| 示例                                    | 功能                                                   |
| ------------                            | ------------                                           |
| show dir                                | 查看当前gdb搜索源码的路径                              |
| dir dirname                             | 添加新的源码路径到搜索路径，多个时用冒号分开           |
| display                                 | 自动打印调试信息                                       |
| undisplay                               | 取消自动打印调试信息                                   |
| watch                                   | 监控断点，当监控变量或内存的内容变化时停止运行         |
| tb                                      | 临时断点                                               |
| x/nfu addr                              | 打印内存块，n: 单元数量，f: 输出格式(bodx, 2-8-10-16进制)，u: 单元长度(bhwg, 1-2-4-8字节) |
| attach PID                              | 附上PID进程并进入调试                                  |
| set substitute-path from\_path to\_path | 映射源码搜索路径中的部分string为其他string             |
| set scheduler-locking off               | 执行所有thread                                         |
| set scheduler-locking on                | 只执行当前thread                                       |
| set scheduler-locking step              | 单步时，除了next过一个函数的情况以外，只执行当前thread |

gdb调试时输出下面信息：
```
Program received signal SIGTRAP, Trace/breakpoint trap
```
原因是编译时和调试时的动态库版本不匹配，编译时使用了devtoolset-7中的gcc-7.3.1，但调试时LD_LIBRARY_PATH没有添加devtoolset-7动态库路径，gdb仍然使用系统默认的gcc-4.8.5动态库，在LD_LIBRARY_PATH添加devtoolset-7动态库路径即可解决


## core文件查看

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
## 重定向已启动进程的stdout和stderr

```
gdb -p $PID
(gdb) p dup2(open("/dev/null", 0), 1)  # 1表示stdout，具体路径为/proc/$PID/fd/1
(gdb) p dup2(open("/dev/null", 0), 2)  # 2表示stderr，具体路径为/proc/$PID/fd/2
(gdb) detach
(gdb) quit
```


## PDB

查看代码 l 查看当前位置前后11行代码，符号-->标示当前位置

查看代码 ll 查看当前函数或框架的所有源代码

查看断点 b/tbreak 查看断点/临时断点

设置断点 b/tbreak filename:lineno 设置断点/临时断点

设置断点 b/tbreak functionname 设置断点/临时断点

清除断点 cl 清除所有断点（包括临时断点）

清除断点 cl filename:lineno 清除指定断点

清除断点 cl bpnumber [bpnumber ...] 清除指定断点

调试 s 执行下一行（能够进入函数体）

调试 n 执行下一行（不会进入函数体）

调试 r 执行下一行（在函数中时会直接执行到函数返回处）

调试 c 持续执行，直到遇到一个断点

调试 unt lineno 持续执行直到运行到指定行（或遇到断点）

调试 j lineno 直接跳转到指定行（被跳过的代码不执行）

查看变量值 p expression 打印变量值

查看类对象的所有成员变量值 p obj.__dict__

查看函数参数 a 在函数中时打印函数的参数和参数的值

打印变量类型 whatis expression 打印表达式的类型，常用来打印变量值

启动交互式解释器 interact 启动一个python的交互式解释器，使用当前代码的全局命名空间（使用ctrl+d返回pdb）

打印堆栈信息 w 最新的帧在最底部，箭头表示当前帧。

启动 python3 -m pdb filename.py 非侵入式方法（不修改源代码，在命令行下直接运行）

启动 import pdb;pdb.set_trace() 侵入式方法（需要在被调试代码中添加一行代码，然后正常运行代码）

退出 q 退出pdb
