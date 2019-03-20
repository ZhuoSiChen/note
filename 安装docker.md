1、查看内核版本,Docker 要求 CentOS 系统的内核版本高于 3.10,否则就升级内核

> [root@localhost ~]# uname -r
>
> 3.10.0-862.el7.x86_64

2、确保yum包更新到最新

> [root@localhost ~]# yum update

3、安装依赖

> [root@localhost ~]# yum install -y yum-utils device-mapper-persistent-data lvm2

4、设置yum源

> [root@localhost ~]# yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
>
> Loaded plugins: fastestmirror
>
> adding repo from: https://download.docker.com/linux/centos/docker-ce.repo
>
> grabbing file https://download.docker.com/linux/centos/docker-ce.repo to /etc/yum.repos.d/docker-ce.repo
>
> repo saved to /etc/yum.repos.d/docker-ce.repo

5、安装docker

> [root@localhost ~]# yum install docker-ce

6、启动docker

> [root@localhost ~]# systemctl start docker

7、加入开机启动项

> [root@localhost ~]# systemctl enable docker
>
> Created symlink from /etc/systemd/system/multi-user.target.wants/docker.service to /usr/lib/systemd/system/docker.service.

8、验证docker是否安装成功

> [root@localhost ~]# systemctl enable docker
>
> Created symlink from /etc/systemd/system/multi-user.target.wants/docker.service to /usr/lib/systemd/system/docker.service.
>
> [root@localhost ~]# docker version
>
> Client:
>
>  Version:      18.03.1-ce
>
>  API version:  1.37
>
>  Go version:   go1.9.5
>
>  Git commit:   9ee9f40
>
>  Built:        Thu Apr 26 07:20:16 2018
>
>  OS/Arch:      linux/amd64
>
>  Experimental: false
>
>  Orchestrator: swarm
>
> Server:
>
>  Engine:
>
>   Version:      18.03.1-ce
>
>   API version:  1.37 (minimum version 1.12)
>
>   Go version:   go1.9.5
>
>   Git commit:   9ee9f40
>
>   Built:        Thu Apr 26 07:23:58 2018
>
>   OS/Arch:      linux/amd64
>
>   Experimental: false
>
> [root@localhost ~]#

