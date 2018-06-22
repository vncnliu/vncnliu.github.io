docker cp F:/download/jdk-8u51-linux-x64.gz 745b66cd78de:/tmp
$ sudo docker run -d -p 5000:5000 training/webapp python app.py

开启远程连接
[root@localhost ~]# more /etc/docker/daemon.json
{
"registry-mirrors": [
    "https://registry.docker-cn.com"
  ],
"hosts": [
    "tcp://0.0.0.0:2375",
    "unix:///var/run/docker.sock"
  ]
}

maven插件根据dockerfile构建 失败 windows 下设置DOCKER_HOST无效
可使用命令行$ DOCKER_HOST=tcp://192.168.1.43:2375 mvn com.spotify:dockerfile-maven-plugin:1.3.6:build -f pom.xml

.停用全部运行中的容器:
docker stop $(docker ps -q)

HTTP / HTTPS代理
泊坞窗守护程序使用HTTP_PROXY，HTTPS_PROXY以及NO_PROXY环境变量在其启动环境来配置HTTP或HTTPS代理的行为。您不能使用该daemon.json文件配置这些环境变量。
此示例覆盖默认docker.service文件。
如果您位于HTTP或HTTPS代理服务器的后面（例如在公司设置中），则需要将此配置添加到Docker systemd服务文件中。
	1. 为docker服务创建一个systemd放置目录：
$ sudo mkdir -p /etc/systemd/system/docker.service.d
	2. 创建一个名为的文件/etc/systemd/system/docker.service.d/http-proxy.conf ，添加HTTP_PROXY环境变量：
[Service]
Environment="HTTP_PROXY=http://proxy.example.com:80/"

或者，如果您位于HTTPS代理服务器的后面，请创建一个名为的文件 /etc/systemd/system/docker.service.d/https-proxy.conf以添加HTTPS_PROXY环境变量：
[Service]
Environment="HTTPS_PROXY=https://proxy.example.com:443/"
	3. 如果您有内部Docker注册表，您需要联系而无需代理，则可以通过NO_PROXY环境变量指定它们：
[Service]
Environment="HTTP_PROXY=http://proxy.example.com:80/" "NO_PROXY=localhost,127.0.0.1,docker-registry.somecorporation.com"

或者，如果您位于HTTPS代理服务器的后面：
[Service]
Environment="HTTPS_PROXY=https://proxy.example.com:443/" "NO_PROXY=localhost,127.0.0.1,docker-registry.somecorporation.com"
	4. 冲洗更改：
$ sudo systemctl daemon-reload
	5. 重新启动Docker：
$ sudo systemctl restart docker
	6. 确认配置已加载：
$ systemctl show --property=Environment docker
Environment=HTTP_PROXY=http://proxy.example.com:80/

或者，如果您位于HTTPS代理服务器的后面：
$ systemctl show --property=Environment docker
Environment=HTTPS_PROXY=https://proxy.example.com:443/


概观
包含所需启动选项而不编辑systemd单元文件的最佳方法是使用systemd插件文件。

解析度
完成这些步骤后，您将启用dockerd的远程API，无需编辑systemd单元文件：

/etc/systemd/system/docker.service.d/startup_options.conf用下面的内容创建一个文件：

# /etc/systemd/system/docker.service.d/override.conf
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -H fd:// -H tcp://0.0.0.0:2376
注： -H标志将dockerd绑定到侦听套接字，可以是Unix套接字或TCP端口。您可以指定多个-H标志以绑定到多个套接字/端口。默认-H fd：//使用systemd的套接字激活功能来引用/lib/systemd/system/docker.socket。

重新加载单位文件：

$ sudo systemctl daemon-reload
用新的启动选项重新启动docker守护进程：

$ sudo systemctl restart docker.service
确保任何有权访问TCP侦听套接字的人都是受信任的用户，因为对docker守护进程的访问权限等同于root。

其他文档
https://docs.docker.com/engine/security/
什么是Docker
