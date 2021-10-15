Git 常见问题

下载失败

 gnutls_handshake() failed: The TLS connection was non-properly terminated.

解决方法：

重置代理

git config --global  --unset https.https://github.com.proxy  
git config --global  --unset http.https://github.com.proxy 