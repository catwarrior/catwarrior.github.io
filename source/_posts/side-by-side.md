# Azure
## Azure website whitelist
``` xml
<system.webServer>
    <security>
      <ipSecurity allowUnlisted="false" denyAction="NotFound">
        <add allowed="true" ipAddress="60.247.xx.xx" subnetMask="255.0.0.0"/>
      </ipSecurity>
    </security>
 </system.webServer>
```
## Azure web job.
 SDK: `Microsoft.Azure.WebJobs` `Microsoft.Azure.WebJobs.Extensions`
 
### Sample:
entry point
``` cs
public static void Main()
        {
            var config = new JobHostConfiguration();

            if (config.IsDevelopment)
            {
                config.UseDevelopmentSettings();
            }
             
            config.UseTimers(); // timer trigger which is defined in SDK extensions.

            JobHost host = new JobHost(config);
            host.RunAndBlock();
        }
```
QueueTrigger
``` cs
   public static void BindToQueueXXXXXX([QueueTrigger("xxxx")] XObject obj, TextWriter log)
        {
           // do stuff here
            log.Write("balabala ");
        }
```
QueueTrigger
``` cs
   public static void BindToBlobsXXXXX([BlobTrigger("xxxx")] XObject obj, TextWriter log)
        {
           // do stuff here
            log.Write("balabala ");
        }
```
## Redis cache
For better performance
``` cs
private static ConnectionMultiplexer connection;
private static ConnectionMultiplexer Connection 
{
    get
    {
        if(connection == null || !connection.IsConnected)
        {
            connection = ConnectionMultiplexer.Connect("yhdcache0.redis.cache.windows.net,ssl=true,password=...");
        }
        return connection;
    }
}
```
# DSL in C# Try/NoException

``` cs
   Func<Action,Action> NoException = (action) =>
   {
       Action wapper = () =>
       {
           try
           {
               action.Invoke();
           }
           catch (Exception ex)
           {
 
               Console.WriteLine(ex);
           }
       };
 
       return wapper;
   };
```
# Docker
## play with boot2docker 
```
boot2docker delete
boot2docker init

boot2docker start

docker run hello-world
``` bash

[a good place show how to use docker in everyday life]https://hub.docker.com/r/tutum/lamp/
[create dev environment with docker file]http://www.open-open.com/lib/view/open1422533340611.html
###  查看docker信息（version、info）
``` bash
# 查看docker版本  
$docker version  
# 显示docker系统的信息  
$docker info 
```
###  对image的操作（search、pull、images、rmi、history）
``` bash
# 检索image  
$docker search image_name  
  
# 下载image  
$docker pull image_name  
  
# 列出镜像列表; -a, --all=false Show all images; --no-trunc=false Don't truncate output; -q, --quiet=false Only show numeric IDs  
$docker images  
  
# 删除一个或者多个镜像; -f, --force=false Force; --no-prune=false Do not delete untagged parents  
$docker rmi image_name  
  
# 显示一个镜像的历史; --no-trunc=false Don't truncate output; -q, --quiet=false Only show numeric IDs  
$docker history image_name  
```
###  启动容器（run）
docker容器可以理解为在沙盒中运行的进程。这个沙盒包含了该进程运行所必须的资源，包括文件系统、系统类库、shell 环境等等。但这个沙盒默认是不会运行任何程序的。你需要在沙盒中运行一个进程来启动某一个容器。这个进程是该容器的唯一进程，所以当该进程结束的时候，容器也会完全的停止。

``` bash
# 在容器中运行"echo"命令，输出"hello word"  
$docker run image_name echo "hello word"  
  
# 交互式进入容器中  
$docker run -i -t image_name /bin/bash  
  
# 在容器中安装新的程序  
$docker run image_name apt-get install -y app_name  
```
Note：  在执行apt-get 命令的时候，要带上-y参数。如果不指定-y参数的话，apt-get命令会进入交互模式，需要用户输入命令来进行确认，但在docker环境中是无法响应这种交互的。apt-get 命令执行完毕之后，容器就会停止，但对容器的改动不会丢失。

### 查看容器（ps）
``` bash
# 列出当前所有正在运行的container  
$docker ps  
# 列出所有的container  
$docker ps -a  
# 列出最近一次启动的container  
$docker ps -l  
```

### 保存对容器的修改（commit）
``` bash
# 保存对容器的修改; -a, --author="" Author; -m, --message="" Commit message  
$docker commit ID new_image_name 
```

### 对容器的操作（rm、stop、start、kill、logs、diff、top、cp、restart、attach）
``` bash
# 删除所有容器  
$docker rm `docker ps -a -q`  
  
# 删除单个容器; -f, --force=false; -l, --link=false Remove the specified link and not the underlying container; -v, --volumes=false Remove the volumes associated to the container  
$docker rm Name/ID  
  
# 停止、启动、杀死一个容器  
$docker stop Name/ID  
$docker start Name/ID  
$docker kill Name/ID  
  
# 从一个容器中取日志; -f, --follow=false Follow log output; -t, --timestamps=false Show timestamps  
$docker logs Name/ID  
  
# 列出一个容器里面被改变的文件或者目录，list列表会显示出三种事件，A 增加的，D 删除的，C 被改变的  
$docker diff Name/ID  
  
# 显示一个运行的容器里面的进程信息  
$docker top Name/ID  
  
# 从容器里面拷贝文件/目录到本地一个路径  
$docker cp Name:/container_path to_path  
$docker cp ID:/container_path to_path  
  
# 重启一个正在运行的容器; -t, --time=10 Number of seconds to try to stop for before killing the container, Default=10  
$docker restart Name/ID  
  
# 附加到一个运行的容器上面; --no-stdin=false Do not attach stdin; --sig-proxy=true Proxify all received signal to the process  
$docker attach ID 
```
Note： attach命令允许你查看或者影响一个运行的容器。你可以在同一时间attach同一个容器。你也可以从一个容器中脱离出来，是从CTRL-C

### 保存和加载镜像（save、load）
当需要把一台机器上的镜像迁移到另一台机器的时候，需要保存镜像与加载镜像。
``` bash
# 保存镜像到一个tar包; -o, --output="" Write to an file  
$docker save image_name -o file_path  
# 加载一个tar包格式的镜像; -i, --input="" Read from a tar archive file  
$docker load -i file_path  
  
# 机器a  
$docker save image_name > /home/save.tar  
# 使用scp将save.tar拷到机器b上，然后：  
$docker load < /home/save.tar  
```

### 登录registry server（login）
``` bash
# 登陆registry server; -e, --email="" Email; -p, --password="" Password; -u, --username="" Username  
$docker login  
``` 

### 根据Dockerfile 构建出一个容器
``` bash
#build  
      --no-cache=false Do not use cache when building the image  
      -q, --quiet=false Suppress the verbose output generated by the containers  
      --rm=true Remove intermediate containers after a successful build  
      -t, --tag="" Repository name (and optionally a tag) to be applied to the resulting image in case of success  
$docker build -t image_name Dockerfile_path  
```

## useful command of docker
``` bash
# start a named container, run a bash
docker run -i -t --name gst tutum/lamp:latest /bin/bash

# remove the container after exit.
# or with -d, run it as a service.
docker run -i -t --rm tutum/lamp

# run additional command on runing container.
docker exec -i -t gst bash

# The -a option to docker ps displays all containers that are currently running or that have exited.
docker ps -a

# You can use docker start to restart a stopped container. After reattaching to it, the contents remain unchanged from the last time that you used the container.
docker start -a -i gst
```

## Docker compose
https://docs.docker.com/compose
http://docs.oracle.com/cd/E52668_01/E54669/html/section_vn2_l2z_fp.html
http://www.open-open.com/lib/view/open1435671611966.html

## Docker file
The instruction keywords define how to create the image:

ADD
Copy the files my.cnf and run.sh from the /var/docker_projects/mymod/mysql directory to /etc/my.cnf and /opt/run.sh in the container.

ENTRYPOINT
Specify that the container always runs /opt/run.sh.

ENV
Define the web proxy in the build environment (as an alternative to modifying /etc/yum.conf).

FROM
Define oraclelinux:6.6 as a basis for the new image.

RUN
Install the mysql-server package and make the /opt/run.sh script executable.

``` bash
# Dockerfile for Rethinkdb 
# http://www.rethinkdb.com/

FROM ubuntu

MAINTAINER Michael Crosby <michael@crosbymichael.com>

RUN echo "deb http://archive.ubuntu.com/ubuntu precise main universe" > /etc/apt/sources.list
RUN apt-get update
RUN apt-get upgrade -y

RUN apt-get install -y python-software-properties
RUN add-apt-repository ppa:rethinkdb/ppa
RUN apt-get update
RUN apt-get install -y rethinkdb

# Rethinkdb process
EXPOSE 28015
# Rethinkdb admin console
EXPOSE 8080

# Create the /rethinkdb_data dir structure
RUN /usr/bin/rethinkdb create

ENTRYPOINT ["/usr/bin/rethinkdb"]

CMD ["--help"]
```

# Linqpad
[Some wonderful examples](http://linqpadsamples.codeplex.com/)

# Toolbox
`Linqpad` `C#` `F#` 

`emacs` `autohotkey`

# .Net
[Common question for C#/.NET](http://www.c-sharpcorner.com/UploadFile/1492b1/C-Sharp-and-Asp-Net-question-and-answersall-in-one/)

[Awesome dotnet](https://github.com/catwarrior/awesome-dotnet)

# Nuget
ex:
`Install-Package Microsoft.AspNet.WebApi -Version 5.2.2`
`Update-Package Microsoft.AspNet.WebApi -reinstall`

# Fsharp
`paket` `fake` `Argu - cli argument parsing`
https://github.com/fsprojects
http://fssnip.net/

# Install all stuff with one click
```bash
START http://boxstarter.org/package/nr/url?https://raw.githubusercontent.com/catwarrior/garage/master/install.txt
```

# Windows command
``` cs 
#Create directory junction 
System.Diagnostics.Process.Start("cmd.exe", "/C mklink /J \"C:\\ProgramData\\xxx\\.\" \"C:\\yyy\"");
```

# windows installer
`log` http://www.msifaq.com/a/1022.htm

# Play with router
[install python on openrt](https://blog.phpgao.com/xiaomi_router_python.html)
[sync with baidupan](http://www.syncy.cn/)
