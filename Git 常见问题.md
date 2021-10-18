Git 常见问题
关闭VPN

下载失败

 gnutls_handshake() failed: The TLS connection was non-properly terminated.

解决方法：

重置代理

git config --global  --unset https.https://github.com.proxy  
git config --global  --unset http.https://github.com.proxy 
sudo apt-get update
sudo apt-get install build-essential fakeroot dpkg-dev libcurl4-openssl-dev
sudo apt-get build-dep git
mkdir ~/git-openssl
cd ~/git-openssl
sudo apt-get source git
sudo dpkg-source -x git_1.9.2-1.dsc
cd git-1.9.2
注意上面的1.9.2要按照你自己的版本文件名进行修改
vi debian/control
找到libcurl4-gnutls-dev，替换为libcurl4-openssl-dev
sudo dpkg-buildpackage -rfakeroot -b
如果fail on test，可以将文件debian/rules中的TEST=test注释掉
最后sudo dpkg -i ../git_1.9.2-1_amd64.deb安装git即可。