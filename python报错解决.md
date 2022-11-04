# python报错记录

1. 详细错误信息如下：

```bash
CondaSSLError: OpenSSL appears to be unavailable on this machine.
OpenSSL is required to download and install packages.
```

解决方案：

```bash
# 第一步配置`anaconda`的环境变量如下（anaconda安装路径）
D:\anaconda3
D:\anaconda3\Scripts
D:\anaconda3\Library\bin

# 第二步可以考虑更换镜像源
channels:
  - https://mirrors.bfsu.edu.cn/anaconda/pkgs/free/win-64/
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge/win-64/
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/win-64/
show_channel_urls: true
ssl_verify: false
```

