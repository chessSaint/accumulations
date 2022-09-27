# linux命令大全
本篇文档用于记录日常使用中遇到的Linux命令用法。

## Linux命令之脚本调试
`set -x`与`set +x`指令用于脚本调试。`set`是把它下面的<font color=red>命令</font>打印到屏幕。其中，`set -x`是开启打印；`set +x`是关闭打印。
```bash
set -x            # activate debugging from here

set +x            # stop debugging from here
```