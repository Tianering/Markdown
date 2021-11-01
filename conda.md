conda无法更新下载，SOLVING ENVIRONMENT: FAILED WITH INITIAL FROZEN SOLVE. RETRYING WITH FLEXIBLE SOLVE

conda update -n base conda
conda update --all





conda 国内源
查看当前源
conda config --show-sources
添加源
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/  #清华源停服 
腾讯云
conda config --add channels https://mirrors.cloud.tencent.com/anaconda/pkgs/free/
conda config --add channels https://mirrors.cloud.tencent.com/anaconda/pkgs/main/
conda config --set show_channel_urls yes
换回默认源
conda config --remove-key channels



创建pytorch环境

https://pytorch.org/get-started/previous-versions/



pip清华

### 临时使用

```
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple some-package
```

注意，`simple` 不能少, 是 `https` 而不是 `http`

### 设为默认

升级 pip 到最新的版本 (>=10.0.0) 后进行配置：

```
pip install pip -U
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```

如果您到 pip 默认源的网络连接较差，临时使用本镜像站来升级 pip：

```
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple pip -U
```

