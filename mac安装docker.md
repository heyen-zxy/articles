# docker搭建手册(mac)

参考文章： [http://www.phperz.com/article/14/1209/40174.html](http://www.phperz.com/article/14/1209/40174.html)

### 下载docker安装包
[https://github.com/boot2docker/osx-installer/releases](https://github.com/boot2docker/osx-installer/releases)上下载最新的release版本，我们直接下载pkg安装包，双击打开安装。

### 配置
iterm中执行` boot2docker init `

如果报错如下：

error in run: Failed to download ISO image: Get https://github-cloud.s3.amazonaws.com/releases

解决方法：

下载boot2docker.iso 将其复制到 ~/.boot2docker/
boot2docker.iso 再执行 ` boot2docker init `

下载链接：[https://github.com/boot2docker/boot2docker/releases](https://github.com/boot2docker/boot2docker/releases])

### 启动
` boot2docker up `或` boot2docker start `启动docker 服务端,

	To connect the Docker client to the Docker daemon, please set:
	    export DOCKER_TLS_VERIFY=1
	    export DOCKER_HOST=tcp://192.168.59.103:2376
	    export DOCKER_CERT_PATH=/Users/zxypow/.boot2docker/certs/boot2docker-vm
	
	Or run: `eval "$(boot2docker shellinit)"`

192.168.59.103就是我们访问docker服务器的地址

根据提示执行 ` eval "$(boot2docker shellinit)" `

这样我们就可以通过 `boot2docker ssh ` 访问docker控制台了

docker教程 [http://dockone.io/article/102](http://dockone.io/article/102)