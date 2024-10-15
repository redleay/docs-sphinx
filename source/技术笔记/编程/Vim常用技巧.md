# Vim常用技巧

## 常用快捷键

| 快捷键   | 作用 |
| ----     | ---- |
| CTRL+]   | 跳转tag |
| CTRL+w+] | 新窗口跳转tag |
| CTRL+o   | 返回tag |
| CTRL+t   | 返回tag |

## 模式匹配

```
\{n,m} Matches n to m of the preceding atom, as many as possible
\{n} Matches n of the preceding atom
\{n,} Matches at least n of the preceding atom, as many as possible
\{,m} Matches 0 to m of the preceding atom, as many as possible
\{} Matches 0 or more of the preceding atom, as many as possible (like *)
*/\{-*
\{-n,m} matches n to m of the preceding atom, as few as possible
\{-n} matches n of the preceding atom
\{-n,} matches at least n of the preceding atom, as few as possible
\{-,m} matches 0 to m of the preceding atom, as few as possible
\{-} matches 0 or more of the preceding atom, as few as possible
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

| 命令 | 作用 |
| ---- | ---- |
| :%sort         | 排序 |
| :%sort!        | 倒序排序 |
| :%sort n       | 按数字大小排序 |
| :%sort u       | 排序时去除重复行 |
| :%sort! n      | 按照数字倒序排序 |
| :%!shuf        | 随机排序 |
| :%!sort -R     | 随机排序 |

删除重复行
:sort
:g/^\(.\+\)$\n\1/d

:%s/\(.\+\)\n\1/\1/g

## 文本对比

启动对比命令
```
vimdiff FILE_LEFT FILE_RIGHT
vimdiff -d FILE_LEFT FILE_RIGHT # 左右竖屏
vimdiff -o FILE_LEFT FILE_RIGHT # 上下横屏
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

