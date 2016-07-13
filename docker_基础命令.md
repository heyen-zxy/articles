# docker 基础命令
` boot2docker ip` 查看当前启动的docker的ip，在docker外执行


` docker images ` 查看所有镜像，docker会默认给image添加一个id

	REPOSITORY            TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
	zxyheyen/hello_work   latest              357f88f457c8        5 seconds ago       122 MB
	zxyheyen/test         latest              93f930a32c1d        2 weeks ago         1.113 MB
	ubuntu                latest              8e5b7248472b        4 weeks ago         122 MB
	<none>                <none>              511136ea3c5a        3 years ago         0 B

`docker ps -a` 查看所有容器，同样docker会给容器添加默认id，如果你未给容器起名字

	CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                      PORTS               NAMES
	032e6eb8794a        ubuntu              "echo hello work"   3 minutes ago       Exited (0) 3 minutes ago                        agitated_feynman
	a1921760d271        ubuntu              "/bin/bash"         17 minutes ago      Exited (0) 17 minutes ago                       ecstatic_varahamihira
	034d590e320d        ubuntu              "/bin/bash"         16 hours ago        Exited (0) 16 hours ago                         small_snyder
	
`docker pull` 拉取远程镜像，和github一样docker也有一个远程的镜像仓库 [https://hub.docker.com](https://hub.docker.com)	

`docker login --username=username --password=password --email=email` 在docker容器登录远程仓库

`docker commit contain username/image` 将某个容器生成或更新到一个镜像，容器名可以用简写id的前几位代替，只要他是唯一的,image名必须带有username，这样我们才能将其推送到我们自己的远程仓库

如上述的容器： `docker commit 032 zxyheyen/hello_work`

`docker push zxyheyen/hello_work` 将我们本地的镜像推送到远程仓库

`docker logs contain_id` 查看某个容器当前状态

`docker rmi image` 删除某个镜像，这里的image可以用简写的image_id，也可以用镜像名

`docker rm contain_id` 删除某个容器

`docker search image-name` 和ruby的`gem search`一样

`docker run image-name` 运行某个镜像，运行一个镜像就会产生一个容器

`docker run -i -t <image> /bin/bash` 使用image创建container并进入交互模式, login shell是/bin/bash

`docker run -i -t -p <host_port:contain_port> `将container的端口映射到宿主机的端口







