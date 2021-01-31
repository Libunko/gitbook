# docker note

## docker run 和 start 的区别

- docker run

    docker run 只在第一次运行时使用，将镜像放到容器中，以后再次启动这个容器时，只需要使用命令 docker start  即可。 docker run 相当于执行了两步操作：将镜像放入容器中（ docker create ），然后将容器启动，使之变成运行时容器（docker start）。

- docker start
     docker start 的作用是，重新启动已存在的镜像。也就是说，如果使用这个命令，我们必须事先知道这个容器的ID，或者这个容器的名字，我们可以使用 docker ps 找到这个容器的信息。
     
- 查看所有容器

     而要显示出所有容器，包括没有启动的，可以使用命令 docker ps -a

- 重命名

    docker rename  old_name new_name 这个容器命名。再次启动或停止容器时，就可以直接使用这个名字。

- 启停 docker [stop]|[start]  name

