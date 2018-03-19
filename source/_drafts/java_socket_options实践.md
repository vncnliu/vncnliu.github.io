###### SO_KEEPALIVE
此参数用于设置是否启用TCP自带的心跳检测机制，在JAVA中只有socket提供设置方法
```java
socket.setKeepAlive(true);
```
需注意的是此参数应在服务端设置，我测试时只在客户端设置并不生效，而只在服务端设置是可以的
此外在linux中有三个系统参数于此相关
* tcp_keepalive_time
在TCP保活打开的情况下，最后一次数据交换到TCP发送第一个保活探测包的间隔，即允许的持续空闲时长，或者说每次正常发送心跳的周期，默认值为7200s（2h）。
* tcp_keepalive_probes
在tcp_keepalive_time之后，没有接收到对方确认，继续发送保活探测包次数，默认值为9（次）
* tcp_keepalive_intvl
在tcp_keepalive_time之后，没有接收到对方确认，继续发送保活探测包的发送频率，默认值为75s。

既然是实践当然要有看得见的东西
{% asset_img wireshark-01.png wireshark packs %}
要得到上图首先要修改tcp_keepalive_time的值，默认值实在是太大了
linux下执行如下代码临时更改
```shell
echo 10 >> /proc/sys/net/ipv4/tcp_keepalive_time
```
