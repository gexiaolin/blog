---
title: sublime启用vim模式
date: 2019-04-30 18:33:32
categories: 
    - 前端开发
tags:
    - sublime
---

# sublime 启用vim模式
## 什么是vim

> Vim是从vi发展出来的一个文本编辑器。其代码补完、编译及错误跳转等方便编程的功能特别丰富，在程序员中被广泛使用。和Emacs并列成为类Unix系统用户最喜欢的编辑器。    
> --维基百科

---

## sublime 启用标准vim模式方法
sublime默认关闭vim模式，打开方法如下：

```
菜单中打开：Preferences -> Setting - User
找到"ignored_packages"字段，将数组中的"Vintage"项删除
```
现在已经启用了vim模式，可以按“Esc”退出编辑模式，进入vim模式。
## vim模式启用热键更改
如果你觉得esc启用vim模式不顺手（比如我），可以更改启动热键，方法如下：

```
菜单中打开：Preferences -> Key Bindings

不建议更改左侧默认热键，我们可以在右侧user文件内覆盖原配置：
[
    { "keys": ["j", "j"], "command": "exit_insert_mode",
        "context":
        [
            { "key": "setting.command_mode", "operand": false },
            { "key": "setting.is_widget", "operand": false }
        ]
    }
]

上面的配置会把j+j(两下j)设置为vim的启动热键，这里你可以设置成自己顺手的启动键位。
```
## 常用vim功能键
```
j: 下移一行
k: 上移一行
h: 左移一格
l: 右移一格
v: 配合hjkl等跳转键进行选中
a: 在当前字符后进入insert模式
A: 在当前行的结尾处进入insert模式
i: 在当前字符前进入insert模式
o: 向下插入空行并进入insert模式
O: 向上插入空行并进入insert模式
d: 删除选中字符（v）
dd: 删除行
3dd: 删除3行
dgg: 删除文件起始至当前行的内容
dG: 删除当前行至文件结束的内容
x: 删除当前字符
y: 复制
P: 当前字符前粘贴
p: 当前字符后粘贴
gu: 切换为小写
gU: 切换为大写
g~: 大小写转换
<: 向前缩进
>: 向后缩进
b: 回退一个单词
B: 回退到上一个空格
5b: 回退5个字符
w: 向前一个单词
W: 向前到下一个空格
5w: 向前5个字符
G: 跳到文件结尾处
gg: 跳到文件起始处
0: 跳到当前行的起始处
$: 跳到当前行的结尾处
{: 跳转到上一个段落
}: 跳转到下一个段落
```
vim还有很多需要记忆的功能键位，以上只是开发中比较常用的一部分，所以学习（记忆）曲线比较陡峭。如果能熟练使用，对于高(tong)效(shi)开(zhuang)发(bi)有很大的帮助。

以上，感谢阅读。

