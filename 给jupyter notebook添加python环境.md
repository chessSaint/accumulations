# 给jupyter notebook添加python环境
注意：这个教程是在Windows系统上，通过Anaconda的虚拟环境给jupyter notebook添加python环境的。



首先，我们打开prompt终端，下面两个任选一个打开即可。

![image-20221026161750177](https://gitee.com/QishengStudent/figs/raw/master/image-20221026161750177.png)

然后，创建想要的添加的虚拟环境（如果已经准备好虚拟环境了，可以忽略这一步）：

```bash
conda create -n envs_name python=x.x
```

创建好环境后，启动环境：

```bash
activate envs_name
```

然后安装`ipykernel`：

```bash
pip install ipykernel
```

最后，使用命令将内核安装到环境中：

```bash
python -m ipykernel install --user --name=envs_name --display-name envs_name
```



安装好之后，重新打开jupyter notebook，点击新建（new）按钮就可以看到新添加的python了！