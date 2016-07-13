# docker-docker_file
docker基础命令里面介绍了很多的命令，docker_file集结这些命令，自动化创建镜像

以下指令不区分大小写，但约定默认是大写

docker_file 都是以 `FROM`开头的

`FROM image-name` 创建或运行容器要以一个镜像为基础 

`MAINTAINER <author name>` 设置镜像的作者

`RUN 《command》` 在当前镜像基础上执行command

`ADD source path` 复制文件到容器，source是来源路径 path是容器中得路径

`EXPOSE port` 指定容器监听的端口

`ENV key value`设置环境变量

`USER uid` 给运行的镜像设置uid

`WORKDIR /path/to/workdir` 设置工作路径

`VOLUME ["/data"]` 授权访问从容器内到主机上的目录

`CMD ["executable","param1","param2"]` CMD：提供了容器默认的执行命令。 Dockerfile只允许使用一次CMD指令。 使用多个CMD会抵消之前所有的指令，只有最后一个指令生效。

`ENTRYPOINT ["executable", "param1","param2"]` ENTRYPOINT：配置给容器一个可执行的命令，这意味着在每次使用镜像创建容器时一个特定的应用程序可以被设置为默认程序。同时也意味着该镜像每次被调用时仅能运行指定的应用。类似于CMD，Docker只允许一个ENTRYPOINT，多个ENTRYPOINT会抵消之前所有的指令，只执行最后的ENTRYPOINT指令。

