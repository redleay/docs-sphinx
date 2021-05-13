# 编程调试技巧

## GDB

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

查看函数参数 a 在函数中时打印函数的参数和参数的值

打印变量类型 whatis expression 打印表达式的类型，常用来打印变量值

启动交互式解释器 interact 启动一个python的交互式解释器，使用当前代码的全局命名空间（使用ctrl+d返回pdb）

打印堆栈信息 w 最新的帧在最底部，箭头表示当前帧。

启动 python3 -m pdb filename.py 非侵入式方法（不修改源代码，在命令行下直接运行）

启动 import pdb;pdb.set_trace() 侵入式方法（需要在被调试代码中添加一行代码，然后正常运行代码）

退出 q 退出pdb
