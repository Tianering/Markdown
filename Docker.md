容器内libGL.so.1问题

sudo apt update
sudo apt install libgl1-mesa-glx



libgthread-2.0.so.0:

apt-get install libglib2.0-dev


远程连接docker 

作者：刘震
链接：https://zhuanlan.zhihu.com/p/52827335

sudo nvidia-docker run -it -p [host_port]:[container_port](do not use 8888) --name:[container_name] [image_name] -v [container_path]:[host_path] /bin/bash
举个栗子：sudo nvidia-docker run -p 5592:5592 -p 5593:5593 -p 8022:22 --name="liuzhen_tf" -v ~/workspace/liuzhen/remote_workspace:/workspace/liuzhen/remote_workspace -it tensorflow/tensorflow:latest-gpu /bin/bash

首先安装openssh-server:$ apt update
$ apt install -y openssh-server
然后建立一个配置文件夹并进行必要的配置：$ mkdir /var/run/sshd
$ echo 'root:shanyi' | chpasswd
$ sed -i 's/PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
$ sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd
$ echo "export VISIBLE=now" >> /etc/profile
如果连不上SSH服务，可能是某些版本的PermitRootLogin yes默认被注释了，可以使用如下(感谢 @Chenjie Xing 的反馈)：
sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config



Subsystem sftp internal-sftp

重启SSH
$ service ssh restart

$ sudo docker port [your_container_name] 22
$ ssh root@[your_host_ip] -p 8022