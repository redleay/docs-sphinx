# Vim常用技巧

## 打开文件

```
vim -O2 file1 file2     # 在左右排列的两个窗口中同时打开 file1 和 file2
```

## 常用快捷键

| 快捷键   | 作用 |
| ----     | ---- |
| CTRL+]   | 跳转tag |
| CTRL+w+] | 新窗口跳转tag |
| CTRL+t   | 返回，根据tag list |
| CTRL+o   | 返回，根据jump list |

## 模式匹配

元字符
```
. 匹配任意一个字符
\t 匹配<TAB>字符。
\n
\r

[abc] 匹配方括号中的任意一个字符。可以使用-表示字符范围，如[a-z0-9]匹 配小写字母和阿拉伯数字。
[^abc] 在方括号内开头使用^符号，表示匹配除方括号中字符之外的任意字符。

\d 匹配阿拉伯数字，等同于[0-9]。
\x 匹配十六进制数字，等同于[0-9A-Fa-f]。
\w 匹配单词字母，等同于[0-9A-Za-z_]。
\s 匹配空白字符，等同于[ \t]。

\D 匹配阿拉伯数字之外的任意字符，等同于[^0-9]。
\X 匹配十六进制数字之外的任意字符，等同于[^0-9A-Fa-f]。
\W 匹配单词字母之外的任意字符，等同于[^0-9A-Za-z_]。
\S 匹配非空白字符，等同于[^ \t]。
```

数量

```
*  匹配任意个
?  匹配0个或1个
\+ 匹配1个或多个

# as many as possible
\{}     Matches 0 or more (like *)
\{n,m}  Matches n to m
\{n,}   Matches at least n
\{,m}   Matches 0 to m
\{n}    Matches n

# as few as possible
\{-}    matches 0 or more
\{-n,m} matches n to m
\{-n,}  matches at least n
\{-,m}  matches 0 to m
\{-n}   matches n
```

位置

```
$ 匹配行尾
^ 匹配行首
< 匹配单词词首
> 匹配单词词尾
```

转义字符

```
\. 匹配 . 字符。
\* 匹配 * 字符。
\/ 匹配 / 字符。
\ 匹配 \ 字符。
\[ 匹配 [ 字符。
```

逻辑关系

```
\&  与
\|  或
```

## 搜索替换

标记g：表示全局搜索，对每一个匹配结果进行操作，缺省标记g，则只对第一个匹配结果进行操作。
标记c：表示操作前需要进行确认。
标记i：表示大小写不敏感。
标记I：表示大小写敏感。

分组
\(
\)

\U: 将后面的字符转换为大写
\u: 将后面的字符转换为小写
&：匹配的字符
\E：结束大小写转换

```
s/\<[abcd][12]\>/\U&/   " 将a1/b1/c1/d1/a2等转换为A1/B1/C1/D1/A2等大写形式
```

## 寄存器

复制寄存器 `a` 的内容到系统剪贴板
```
:let @+ = @a   " Linux/Windows 用 '+' 寄存器
:let @* = @a   " macOS 通常使用 '*' 寄存器
```

## 配置管理

```
:version
```

## quickfix窗口跳转

| 命令 | 作用 |
| ---- | ---- |
| :col[colder] | 上一个quickfix窗口 |
| :cnewer | 下一个quickfix窗口 |
| :cnewer | 下一个quickfix窗口 |

## 文本排序

### 内置命令:sort

对`[range]`范围，根据每行匹配`{pattern}`之内或之后的文本进行排序（而且是稳定排序）

```
:[range]sort[!] [b][o][n][x][f][i][r][u] [/{pattern}/]
:help :sort # 查看:sort的help
```

| 参数 | 作用 |
| ---- | ---- |
| range | 排序范围，默认为`%`，即对整个文档排序 |
| ! | 反向排序 |
| b | 基于每行的第1个2进制数排序 |
| o | 基于每行的第1个8进制数排序 |
| n | 基于每行的第1个10进制数排序，包含前导的`-` |
| x | 基于每行的第1个16进制数排序，包含前导的`-`，忽略`0x`或`0X` |
| f | 基于每行的第1个浮点数排序，仅当 Vim 编译时支持浮点数时才有效 |
| r | 基于`{pattern}`匹配的内容排序，否则基于匹配之后的内容排序 |
| i | 忽略大小写 |
| u | 只保留完全相同的行的第一行 |

示例：

```
:sort         # 排序
:sort!        # 倒序排序
:sort n       # 按数字大小排序
:sort u       # 排序并去除重复行
:uniq         # 去除重复行
:sort r       # 随机排序
```

删除重复行
```
# 方法1
:sort
:g/^\(.\+\)$\n\1/d
# 方法2
:sort
:%s/\(.\+\)\n\1/\1/g
```

### 外部命令

```
:!sort -R     # 随机排序
:!shuf        # 随机排序
```

## 文本对比

命令行启动对比
```
vimdiff LEFT RIGHT    # 等同于 vim -d LEFT RIGHT
vimdiff -d LEFT RIGHT # 左右竖屏
vimdiff -o LEFT RIGHT # 上下横屏
```

Vim内部一条命令启动
```
:diffsplit RIGHT
:vertical diffsplit RIGHT
```

Vim内部多条命令启动，适合于多个文件相互对比
```
:e FILE1
:diffthis
:vs FILE2   or  :s FILE2
:diffthis
:diffoff    # 对两个窗口分别操作
```

对比操作

| 命令            | 作用 |
| ----            | ---- |
| ]c              | 跳转到下一个差异点 |
| [c              | 跳转到上一个差异点 |
| dp(diff put)    | 将差异点内容从当前文件复制到另一个文件 |
| do(diff obtain) | 将差异点内容从另一个文件复制到当前文件 |
| :diffupdate     | 手工来刷新比较结果 |


## 多行处理

将.KS开始的行到.KE开始的行，作为一个文本块，并在该块内将行尾替换为@@
```
:g/^\.KS/,/^\.KE/-1s/$/@@/
```

将.KS开始的行到.KE开始的行，作为一个文本块，并将该块合并为一行
```
:g/^\.KS/,/^\.KE/j
```

## 其他
输出重定向
```
:redir @a
:map
:redir END
```

查看最后映射<CTRL>F的脚本位置
```
:verbose map <CTRL>F
```

文件类型问题：移除文件显示的`^M`字符
```
:e ++ff=dos
```

## 插件

### Vundle

| 命令 | 作用 |
| ---- | ---- |
| :PluginInstall | 安装插件 |
| :PluginUpdate  | 更新插件 |
| :PluginClean   | 移除插件 |
| :PluginList    | 列出所有插件 |
| :PluginSearch  | 查找插件 |

