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

