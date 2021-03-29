## coturn 配置
作者：康林（kl222@126.com)

### 执行程序
- turnserver: 服务程序
- turnadmin：实际就是turnserver
- turnutils_natdiscovery
- turnutils_peer
- turnutils_uclient
- turnutils_oauth
- turnutils_stunclient

### 配置文件

turnserver.conf



### ICE测试

打开下面地址：
https://webrtc.github.io/samples/src/content/peerconnection/trickle-ice/

分别测试stun和turn服务器，只有relay地址回来的是你的ip才算穿透成功。
