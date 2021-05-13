# VIM使用技巧

quickfix窗口跳转

| 命令 | 作用 |
| ---- | ---- |
| :col[colder] | 上一个quickfix窗口 |
| :cnewer | 下一个quickfix窗口 |
| :cnewer | 下一个quickfix窗口 |

Vundle插件管理

| 命令 | 作用 |
| ---- | ---- |
| :PluginInstall | 安装插件 |
| :PluginUpdate  | 更新插件 |
| :PluginClean   | 移除插件 |
| :PluginList    | 列出所有插件 |
| :PluginSearch  | 查找插件 |

文本排序

| 命令 | 作用 |
| ---- | ---- |
| :%sort         | 排序 |
| :%sort!        | 倒序排序 |
| :%sort n       | 按数字大小排序 |
| :%sort u       | 排序时去除重复行 |
| :%sort! n      | 按照数字倒序排序 |


将.KS开始的行到.KE开始的行，作为一个文本块，并在该块内将行尾替换为@@
```
:g/^\.KS/,/^\.KE/-1s/$/@@/
```

将.KS开始的行到.KE开始的行，作为一个文本块，并将该块合并为一行
```
:g/^\.KS/,/^\.KE/j
```

输出重定向
```
:redir @a
:map
:redir END
```
