# 高级键位

经过前面几章的洗礼，相信你已经对 Vim 有了一个全面的了解，你会发现它不像你想象中的那样难用。现阶段你仍然不会觉得它的好，除了适应的问题外，更多原因是我们需要学习的东西还很多，这一章节除了补充些常用知识外，主要会介绍几个“绝活”让编辑文档更简单。你要做好准备，因为这一章又是需要多练习的一章。

## 保存退出

第一章中我们简单说明保存退出的按键，但是还有很多情况是 `:wq` 处理不了的

**直接退出**

```vim
ZZ
```

就是两个大写的 `Z`，如果没有修改文件直接退出，如果有修改则保存后退出。有点太直接了，说实话并不常用，保存退出很多时候还是小心操作的好。

**强制退出**

如果你修改了写东西，突然不想保存，直接退出是会报错的

```bash
E37: No write since last change (add ! to override)
```

通过提示可以看出，最后的修改没有保存，需要使用 `!` 强制退出。

```vim
:q!
```

**强制保存**

你应该能很自然的想到强制保存也是加上 `!`

```vim
:w!
```

没错，`!` 在 Vim 就是有强制的意思，但是最后能不能保存成功还是要看当前用户对应该文件的权限，所以在 Linux 上编辑文件尽量使用 `sudo` 打开文件。

```bash
$ sudo vim Object.java
```

不过话说回来，假如我忘记了权限的问题，又做了大量的修改，难道要让我放弃掉吗？当然有办法解决。

```vim
:w ! sudo tee %
```

通过该命令可以强制在 Vim 中使用 `sudo` 权限保存

**另存为**

看到这个名称你肯定很熟悉，任何编辑器左上角打开“编辑”按钮都会有这个功能，Vim 的要更方便一点，在保存的后边跟上新文件名即可

```vim
:w newfilename
```

你也可以只保存某几行数据到新文件，比如 1 到 10 行

```vim
:1, 10 w newfilename
```

**写入另一个文件的内容**

如果你在写作的时候，需要将另一个文件的内容直接引用到当前文件，可以试试这个功能

```vim
:r filename
```

然后目标内容就会粘贴到当前光标一下的位置，你可以指定它追加到某一行的下面。

```vim
:2 r filename
```

**暂时离开 Vim**

假如你想要写入另一个文件的内容，但是忘记了文件名，但是又不想退出文件怎么办，我们有办法可以暂时离开 Vim 环境来执行 shell 命令。

```vim
:! ls
```

随后你会看到执行命令的结果，然后回车即可返回 Vim，前期我们借助不到插件的帮忙时，这个功能是非常有用的，不过它只能执行非常基础的命令，`ll` 都会报错。

## 打开文件警告

我们在用 Vim 编辑文档事，它会自动在文件同目录下生一个 `.filename.swp` 文件，它是作为一个缓存存在的，记录了你最后的编辑时刻，如果正常关闭，则会自动删除，如果非常关闭，它就会保存下来。在需要找回文件时它是很有用的，不过再次打开原文件时却会有个麻烦的警报。

下面我们来模拟下非正常退出的场景，再打开 Vim 的情况下，使用 `<c-z>` 组合键将 Vim 放到后台执行。

```bash
[1]+  Stopped                 vim Object.java
```

出现上述提示，即为成功，此时已经返回命令行界面，查看文件列表。

```bash
$ ll -a
-rw-r--r--  1 root root   26820 Apr 10 21:31 Object.java
-rw-r--r--  1 root root   40960 Apr 10 21:22 .Object.java.swp
```

我们可以看到 `.Object.java.swp` 的出现，此时再次打开 `Object.java` 就会报错。

```bash
$ vim Object.java
E325: ATTENTION
Found a swap file by the name ".Object.java.swp"
          owned by: root   dated: Tue Apr 10 21:22:24 2018
         file name: ~root/Object.java
          modified: no
         user name: root   host name: iZ2zefuxiq5o2n1epu4h6rZ
        process ID: 13579 (still running)
While opening file "Object.java"
             dated: Tue Apr 10 21:31:51 2018
      NEWER than swap file!

(1) Another program may be editing the same file.  If this is the case,
    be careful not to end up with two different instances of the same
    file when making changes.  Quit, or continue with caution.
(2) An edit session for this file crashed.
    If this is the case, use ":recover" or "vim -r Object.java"
    to recover the changes (see ":help recovery").
    If you did this already, delete the swap file ".Object.java.swp"
    to avoid this message.

Swap file ".Object.java.swp" already exists!
[O]pen Read-Only, (E)dit anyway, (R)ecover, (Q)uit, (A)bort:
```

这个错误第一次看会很乱，我们只看最后两行。

```bash
Swap file ".Object.java.swp" already exists!
[O]pen Read-Only, (E)dit anyway, (R)ecover, (Q)uit, (A)bort:
```

缓存文件已经存在，问我们想要怎么办，下面的五种提示使用首字母可以进行相应操作，它们的意思分别为

- `[O]pen Read-Only` 我不管，我不管，就要继续打开文件。但此时打开的是只读文件，这种情况我们可以使用上边讲到的强制保存来写入文件，但是如果这是其他人正在打开的文件，看看就退出好了。
- `(E)dit anyway` 依然不管，继续修改文件。此次修改过后需要记得删除 `.swp` 文件，不然下次还会出现。
- `(R)ecover` 使用之前的缓存进行覆盖。同样 `.swp` 文件要手动删除。
- `(Q)uit` Ok，知道有问题，我退出。然后手动删除 `.swp` 文件再进来，一般都会这样操作。
- `(A)bort` 跟 `q` 一个意思，也是退出。
- `(D)elete it` 多出来这个操作，是在异常退出 `Terminal` 后出现，你肯定会有机会碰到。它就是直接删除 `.swp` 的意思，只要你确保缓存没有用了。

倘若你在没有删除 `.swp` 文件的情况下，再次异常退出了 Vim，那当前目录就会生成一个最新的缓存文件 `.swo` 后缀，如此往复会继续生成 `.swn` 文件，你了解这个事情就好。

## 多窗口操作

在《常用模式》章节中，我们简单了解过多窗口操作，还记得吧。

在我的工作中，多窗口操作使用非常频繁，他真的非常方便。

在《键位操作》章节中，我们提到 `yy` 只在当前缓存区生效，跨文件复制数据就是个问题，但是在多窗口中这个问题可以得到解决，他们同属一个缓存区。

**分割窗口**

```vim
:sp filename    " 上下分割窗口
:vsp filename   " 左右分割窗口
```

`:vsp` 分割方式更有用一些，因为如果你开发的时候严格按照每行最多 79 个字符的话，左右分屏刚好可以在不遮挡代码的情况下，对照编辑。

**移动光标**

使用组合键 `<c-w>` 再加上 `h, j, k, l` 任意一个方向键，就可以将光标移动到对应方向的窗口上。

```vim
<c-w>h[j[k[l]]]     " 根据方向键移动光标到该方向的窗口上
```

**调换窗口位置**

在你分窗口时，可能默认的位置并不是你想要的，这时候需要手动调换下位置。

```vim
<c-w>x      " 对调左右或上下两个对应的窗口
```

必须是对应的两个窗口，如果左边一个，右边上下两个，那左右就不能调换。另外还有 `<c-w>r` `<c-w>R` 两个命令，分别为顺时针和逆时针调换窗口，现在的实验效果跟 `<c-w>x` 效果差不多，所以可以不用记忆。

**退出窗口**

```vim
<c-w>q
```

每次用这个键，我都会感觉很蛋疼，他们的位置太近了，操作极其不方便，所以我干脆使用 `:q` 退出

## 多文件编辑

你是不是一直都在用 Vim 打开单个文件，其实他也可以像其他 IDE 一样同时打开多个文件。

**打开多文件**

```bash
$ cp Object.java Object.java.copy
$ vim Object.java Object.java.copy
```

**查看打开的文件列表**

```vim
:files
```

```vim
  1 %a   "Object.java"                  line 1
  2      "Object.java.copy"             line 0
```

通过列表信息显示，当前打开了两个文件。`%a` 标识出现在正在编辑的文件。回车返回。

**文件跳转**

```vim
:n      " 跳到下一个文件
:N      " 跳到上一个文件
```

这里有点不太方便，你可能会想着使用 `:1` `:2` 来跳转文件，但是只会跳到相应的行数。`:n` 会一直跳到下一个文件，到最后一个文件后，只能使用 `:N` 往回跳。

总体来说多文件编辑是有些尴尬的，前期多窗口编辑比它方便，后期也是被各种插件吊打，如果不是要写这个本书，我都快忘了这个操作了，你可以记下来，需要的时候知道怎么用。

## c 的使用

`c` 在开发届一直都代表着老大的地位，在 Vim 中 `c` 的频率可以说是相当高。

在《常用模式》章节中，我们介绍过 `cc` 可以删除当前行并进入 `insert` 模式，`c` 就是替换的意思，使用它来替换掉你想要修改的地方重新输入。

只输入 `c` 是没有效果的，需要加上一下动作

```vim
cc      " 删除当前行，并进入 insert 模式
D       " 删除当前行，并进入 insert 模式
cj      " 删除当前行和下一行，并进入 insert 模式
c10j    " 删除当前行和下面 10 行，并进入 insert 模式
```

除了这些基础操作，配置上 Operator-Pending 映射简直如虎添翼。

```vim
ce      " 删除光标到当前单词结尾，并进入 insert 模式
cw      " 删除光标到当前单词结尾，并进入 insert 模式
cb      " 删除光标到当前单词开始，并进入 insert 模式

ciw     " 删除当前单词，并进入 insert 模式
ci(     " 删除光标所在括号的内容，并进入 insert 模式，适用于 {[(<`'" 等成对的符号
cip     " 删除当前段落，并进入 insert 模式
ct,     " 删除到下一个逗号，并进入 insert 模式，逗号可以换成所有可输入字符
```

我们先不要管什么叫 Operator-Pending 映射，以后我们会单开一个章节讲它，你只需要先照着用起来就好。

另外需要知道的是上面所有操作将 `c` 换成 `v, y, d` 也可以起到相同的选中、复制和删除的效果。

这里需要说一下 `w, e, b` 三个键位，他们可以在行内快速移动光标。

```vim
w           " 移动到下一个单词开头
e           " 移动到单词的结尾
b           " 移动到单词的开头
```

这三个键位配合以前学习的知识，可以更大程度上方便了行内的光标移动。

## 宏

宏是一个非常有趣的概念，它就像一个录影机一样，将你的所有操作都记录下来，然后再一遍一遍的播放出来。`一遍一遍`，恩，你想到了什么？

批量操作，宏可以完成很多你需要一遍一遍有规律的操作。

举个例子，


2018-04-10 写于北京

2018-04-11 最后修改
