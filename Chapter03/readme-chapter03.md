## Chapter 03

### Exercise 3.01: Working with Docker Image Layers

```sh
docker build -t basic-app .[without curl]

docker history basic-app
IMAGE          CREATED          CREATED BY                                      SIZE      COMMENT
6c6f390b01d1   9 seconds ago    /bin/sh -c apk add wget                         2.33MB
0dc3fff5efd8   11 seconds ago   /bin/sh -c apk update                           2.46MB
9c6f07244728   2 months ago     /bin/sh -c #(nop)  CMD ["/bin/sh"]              0B
<missing>      2 months ago     /bin/sh -c #(nop) ADD file:2a949686d9886ac7c…   5.54MB

docker build -t basic-app .[without curl]
...
tep 3/3 : RUN apk add wget
 ---> Using cache
 ---> 6c6f390b01d1
Successfully built 6c6f390b01d1
Successfully tagged basic-app:latest

docker build -t basic-app .[with curl]
docker images
REPOSITORY   TAG       IMAGE ID       CREATED          SIZE
basic-app    latest    c9db6ae1a060   56 seconds ago   12.5MB
<none>       <none>    6c6f390b01d1   3 minutes ago    10.3MB
alpine       latest    9c6f07244728   2 months ago     5.54MB

docker image inspect 6c6f390b01d1
...
"Data": {
                "LowerDir": "/var/snap/docker/common/var-lib-docker/overlay2/771c0ac2a33700a780cb3a4d72dfc8652b2817b445f505c98336c1918c795755/diff:/var/snap/docker/common/var-lib-docker/overlay2/a296d84ce66b23a5aa5a8c4c7baff8792d1dad426cdd92e7932d5b47d87aeb27/diff",
                "MergedDir": "/var/snap/docker/common/var-lib-docker/overlay2/949ac3ef57d303db640b0a6981721e356ac4094a4c0f85b8008ab7a55d1ee89f/merged",
                "UpperDir": "/var/snap/docker/common/var-lib-docker/overlay2/949ac3ef57d303db640b0a6981721e356ac4094a4c0f85b8008ab7a55d1ee89f/diff",
                "WorkDir": "/var/snap/docker/common/var-lib-docker/overlay2/949ac3ef57d303db640b0a6981721e356ac4094a4c0f85b8008ab7a55d1ee89f/work"
        },
...
sudo du -sh /var/snap/docker/common/var-lib-docker/overlay2/
16M     /var/snap/docker/common/var-lib-docker/overlay2/

docker images -a
REPOSITORY   TAG       IMAGE ID       CREATED          SIZE
basic-app    latest    c9db6ae1a060   8 minutes ago    12.5MB
<none>       <none>    6c6f390b01d1   10 minutes ago   10.3MB
<none>       <none>    0dc3fff5efd8   10 minutes ago   8.01MB
alpine       latest    9c6f07244728   2 months ago     5.54MB

docker image prune
WARNING! This will remove all dangling images.
Are you sure you want to continue? [y/N] y
Deleted Images:
deleted: sha256:6c6f390b01d1ac2c31b196628de041a5d6a50ceac463498d8bf9f6a7919bee07
deleted: sha256:37dc70f54bf36c243eaa5073458393460ddf4460a0e998fd7d3061d80be00809

Total reclaimed space: 2.333MB

```

### Exercise 3.02: Increasing Build Speed and Reducing Layers

```sh
docker rmi -f $(docker images -a -q)

docker pull alpine

mkdir ver1
cp Dockerfile_ver1 ver1
cd ver1
mv Dockerfile_ver1 Dockerfile
tar zcvf Dockerfile.tar.gz Dockerfile

time docker build -t basic-app .
Sending build context to Docker daemon  3.072kB
Step 1/10 : FROM alpine
 ---> 9c6f07244728
Step 2/10 : RUN apk update
 ---> Running in 3ba61170b7e0
fetch https://dl-cdn.alpinelinux.org/alpine/v3.16/main/x86_64/APKINDEX.tar.gz
fetch https://dl-cdn.alpinelinux.org/alpine/v3.16/community/x86_64/APKINDEX.tar.gz
v3.16.2-334-gfa0f6c3878 [https://dl-cdn.alpinelinux.org/alpine/v3.16/main]
v3.16.2-336-gda67e2106e [https://dl-cdn.alpinelinux.org/alpine/v3.16/community]
OK: 17037 distinct packages available
Removing intermediate container 3ba61170b7e0
 ---> 4af8e665e38d
Step 3/10 : RUN apk add wget curl
 ---> Running in 6d7b0641f998
(1/8) Installing ca-certificates (20220614-r0)
(2/8) Installing brotli-libs (1.0.9-r6)
(3/8) Installing nghttp2-libs (1.47.0-r0)
(4/8) Installing libcurl (7.83.1-r3)
(5/8) Installing curl (7.83.1-r3)
(6/8) Installing libunistring (1.0-r0)
(7/8) Installing libidn2 (2.3.2-r2)
(8/8) Installing wget (1.21.3-r0)
Executing busybox-1.35.0-r17.trigger
Executing ca-certificates-20220614-r0.trigger
OK: 10 MiB in 22 packages
Removing intermediate container 6d7b0641f998
 ---> c3f318c6bda8
Step 4/10 : RUN wget -O test.txt https://github.com/PacktWorkshops/The-Docker-Workshop/raw/master/Chapter03/Exercise3.02/100MB.bin
 ---> Running in 3473750bbccd
--2022-10-22 12:40:12--  https://github.com/PacktWorkshops/The-Docker-Workshop/raw/master/Chapter03/Exercise3.02/100MB.bin
Resolving github.com (github.com)... 20.200.245.247
Connecting to github.com (github.com)|20.200.245.247|:443... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://raw.githubusercontent.com/PacktWorkshops/The-Docker-Workshop/master/Chapter03/Exercise3.02/100MB.bin [following]
--2022-10-22 12:40:12--  https://raw.githubusercontent.com/PacktWorkshops/The-Docker-Workshop/master/Chapter03/Exercise3.02/100MB.bin
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 185.199.109.133, 185.199.108.133, 185.199.111.133, ...
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|185.199.109.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 104857600 (100M) [application/octet-stream]
Saving to: 'test.txt'

     0K .......... .......... .......... .......... ..........  0% 3.87M 26s
...
102350K .......... .......... .......... .......... ..........100% 6.37M 0s
102400K                                                       100% 0.00 =22s

2022-10-22 12:40:40 (4.65 MB/s) - 'test.txt' saved [104857600/104857600]
...
Step 10/10 : RUN cat Dockerfile
 ---> Running in 8cc6145d3ea4
FROM alpine

RUN apk update
RUN apk add wget curl

RUN wget -O test.txt https://github.com/PacktWorkshops/The-Docker-Workshop/raw/master/Chapter03/Exercise3.02/100MB.bin

CMD [mkdir /var/www/, mkdir /var/www/html/]
#CMD mkdir /var/www/html/

WORKDIR /var/www/html/

COPY Dockerfile.tar.gz /tmp/
RUN tar -zxvf /tmp/Dockerfile.tar.gz -C /var/www/html/
RUN rm /tmp/Dockerfile.tar.gz

RUN cat Dockerfile
Removing intermediate container 8cc6145d3ea4
 ---> 99f804ba62d9
Successfully built 99f804ba62d9
Successfully tagged basic-app:latest

real    0m43.025s
user    0m0.266s
sys     0m2.906s

docker history basic-app
IMAGE          CREATED         CREATED BY                                      SIZE      COMMENT
99f804ba62d9   5 minutes ago   /bin/sh -c cat Dockerfile                       0B
3d1716e8c9b5   5 minutes ago   /bin/sh -c rm /tmp/Dockerfile.tar.gz            0B
7aa86eb2f589   5 minutes ago   /bin/sh -c tar -zxvf /tmp/Dockerfile.tar.gz …   400B
58e5a25bad60   5 minutes ago   /bin/sh -c #(nop) COPY file:57a9e11a5b2730fc…   348B
773f06a2d497   5 minutes ago   /bin/sh -c #(nop) WORKDIR /var/www/html/        0B
85be10171962   5 minutes ago   /bin/sh -c #(nop)  CMD ["/bin/sh" "-c" "[mkd…   0B
9f2700074d5e   5 minutes ago   /bin/sh -c wget -O test.txt https://github.c…   105MB
c3f318c6bda8   5 minutes ago   /bin/sh -c apk add wget curl                    4.45MB
4af8e665e38d   5 minutes ago   /bin/sh -c apk update                           2.46MB
9c6f07244728   2 months ago    /bin/sh -c #(nop)  CMD ["/bin/sh"]              0B
<missing>      2 months ago    /bin/sh -c #(nop) ADD file:2a949686d9886ac7c…   5.54MB

cd ../ver2
docker rmi -f $(docker images -a -q)
docker pull alpine
docker build -t basic-app .

docker history basic-app
IMAGE          CREATED          CREATED BY                                      SIZE      COMMENT
c9cbfddeaaf0   19 seconds ago   /bin/sh -c cat Dockerfile                       0B
50d85cf4a28a   21 seconds ago   /bin/sh -c #(nop) ADD file:57a9e11a5b2730fc7…   400B
2652e6dd297d   21 seconds ago   /bin/sh -c #(nop) WORKDIR /var/www/html/        0B
ad4ed9967e39   21 seconds ago   /bin/sh -c #(nop)  CMD ["/bin/sh" "-c" "-p m…   0B
d3becc16bc0d   22 seconds ago   /bin/sh -c wget -O test.txt https://github.c…   105MB
337bfa182830   43 seconds ago   /bin/sh -c apk update && apk add wget curl      6.91MB
9c6f07244728   2 months ago     /bin/sh -c #(nop)  CMD ["/bin/sh"]              0B
<missing>      2 months ago     /bin/sh -c #(nop) ADD file:2a949686d9886ac7c…   5.54MB
[8 layers]

cd ../ver3
#docker rmi -f $(docker images -a -q)
docker system prune
mv Dockerfile_basic_base Dockerfile
docker build -t basic-base .
mv Dockerfile Dockerfile_basic_base
cp ../Dockerfile .

time docker build -t basic-app .
Sending build context to Docker daemon  4.096kB
Step 1/5 : FROM basic-base
 ---> 2e3b99a48c5f
Step 2/5 : CMD mkdir -p /var/www/html/
 ---> Running in 4bfef3d91436
Removing intermediate container 4bfef3d91436
 ---> 45c009ffc4dc
Step 3/5 : WORKDIR /var/www/html/
 ---> Running in 60c3802518a6
Removing intermediate container 60c3802518a6
 ---> 0d7881543886
Step 4/5 : ADD Dockerfile.tar.gz /var/www/html/
 ---> 607741a18162
Step 5/5 : RUN cat Dockerfile
 ---> Running in 16007cd90612
FROM alpine

RUN apk update
RUN apk add wget curl

RUN wget -O test.txt https://github.com/PacktWorkshops/The-Docker-Workshop/raw/master/Chapter3/3.2Exercise2/100MB.bin

CMD mkdir /var/www/
CMD mkdir /var/www/html/

WORKDIR /var/www/html/

COPY Dockerfile.tar.gz /tmp/
RUN tar -zxvf /tmp/Dockerfile.tar.gz -C /var/www/html/
RUN rm /tmp/Dockerfile.tar.gz

RUN cat Dockerfile

Removing intermediate container 16007cd90612
 ---> 7aec2d25a679
Successfully built 7aec2d25a679
Successfully tagged basic-app:latest

real    0m2.484s
user    0m0.033s
sys     0m0.091s

docker history basic-app
IMAGE          CREATED              CREATED BY                                      SIZE      COMMENT
7aec2d25a679   About a minute ago   /bin/sh -c cat Dockerfile                       0B
607741a18162   About a minute ago   /bin/sh -c #(nop) ADD file:5c2b84ca4cdaea975…   377B
0d7881543886   About a minute ago   /bin/sh -c #(nop) WORKDIR /var/www/html/        0B
45c009ffc4dc   About a minute ago   /bin/sh -c #(nop)  CMD ["/bin/sh" "-c" "mkdi…   0B
2e3b99a48c5f   6 minutes ago        /bin/sh -c wget -O test.txt https://github.c…   105MB
50d0deda73a9   7 minutes ago        /bin/sh -c apk update && apk add wget curl      6.91MB
9c6f07244728   2 months ago         /bin/sh -c #(nop)  CMD ["/bin/sh"]              0B
<missing>      2 months ago         /bin/sh -c #(nop) ADD file:2a949686d9886ac7c…   5.54MB

docker run -it basic-app sh
/var/www/html # vi prod_test_data.txt
...
exit

docker ps -a
CONTAINER ID   IMAGE       COMMAND   CREATED         STATUS                      PORTS     NAMES
673b4ea4712d   basic-app   "sh"      2 minutes ago   Exited (0) 20 seconds ago             infallible_napier

docker commit 673b4ea4712d basic-app-test
sha256:7d192bffa3d932aaf1e0a8b833365bf1ae7df1f6b800bf4072c5c7993d1f6d91

ocker history basic-app-test
IMAGE          CREATED          CREATED BY                                      SIZE      COMMENT
b8e0d2b49c5f   27 seconds ago   sh                                              41B
7aec2d25a679   9 minutes ago    /bin/sh -c cat Dockerfile                       0B
607741a18162   9 minutes ago    /bin/sh -c #(nop) ADD file:5c2b84ca4cdaea975…   377B
0d7881543886   9 minutes ago    /bin/sh -c #(nop) WORKDIR /var/www/html/        0B
45c009ffc4dc   9 minutes ago    /bin/sh -c #(nop)  CMD ["/bin/sh" "-c" "mkdi…   0B
2e3b99a48c5f   14 minutes ago   /bin/sh -c wget -O test.txt https://github.c…   105MB
50d0deda73a9   15 minutes ago   /bin/sh -c apk update && apk add wget curl      6.91MB
9c6f07244728   2 months ago     /bin/sh -c #(nop)  CMD ["/bin/sh"]              0B
<missing>      2 months ago     /bin/sh -c #(nop) ADD file:2a949686d9886ac7c…   5.54MB

docker run basic-app-test cat prod_test_data.txt
Hello World!!

```

### Exercise 3.04: Using the Scratch Image

```sh
docker build -t scratchtest .
docker run scratchtest
docker images scratchtest
docker history scratchtest
IMAGE          CREATED              CREATED BY                                      SIZE      COMMENT
0d0f15af03d3   About a minute ago   /bin/sh -c #(nop)  CMD ["/test"]                0B
9bcf5dbbb4b7   About a minute ago   /bin/sh -c #(nop) ADD file:ee9a9e97da6da2aa1…   872kB
```

### Exercise 3.05: Tagging Docker Images

```sh
# clear up
docker rmi -f $(docker images -a -q)
docker pull busybox
docker images
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
busybox      latest    ff4a8eb070e1   2 weeks ago   1.24MB

docker tag ff4a8eb070e1 new_busybox:ver_1

docker images
REPOSITORY    TAG       IMAGE ID       CREATED       SIZE
busybox       latest    ff4a8eb070e1   2 weeks ago   1.24MB
new_busybox   ver_1     ff4a8eb070e1   2 weeks ago   1.24MB

docker tag new_busybox:ver_1 chomskim/busybox:ver_1.1
docker images

[Exercise3.05 folder]
echo "FROM new_busybox:ver_1" > Dockerfile
docker build -t built_image:ver_1.1.1 .
docker images
REPOSITORY         TAG         IMAGE ID       CREATED       SIZE
chomskim/busybox   ver_1.1     ff4a8eb070e1   2 weeks ago   1.24MB
built_image        ver_1.1.1   ff4a8eb070e1   2 weeks ago   1.24MB
busybox            latest      ff4a8eb070e1   2 weeks ago   1.24MB
new_busybox        ver_1       ff4a8eb070e1   2 weeks ago   1.24MB

```

### Exercise 3.07: Automating Your Image Tagging

```sh
[Exercise3.07 folder]
git init; git add Dockerfile; git commit -m "initial commit"

docker build -t basic-base .

tar zcvf Dockerfile.tar.gz Dockerfile
docker build -t basic-app:$(git log -1 --format=%h) .
...
Successfully tagged basic-app:fe54061

docker build -t basic-app --build-arg GIT_COMMIT=$(git log -1 --format=%h) .
docker inspect -f '{{index .ContainerConfig.Labels "git-commit"}}' basic-app
fe54061

chmod +x build.sh

./build.sh
++ USER=chomskim
++ SERVICENAME=basic-app
+++ cat VERSION
++ version=1.0.0
++ echo 'version: 1.0.0'
version: 1.0.0
++ docker build -t chomskim/basic-app:1.0.0 .
Sending build context to Docker daemon   59.9kB
Step 1/6 : FROM basic-base
 ---> db017ea97c3d
Step 2/6 : CMD mkdir -p /var/www/html/
 ---> Running in 08f7de87a013
Removing intermediate container 08f7de87a013
 ---> 18b8cd74fa85
Step 3/6 : WORKDIR /var/www/html/
 ---> Running in 3aa45abb4d67
Removing intermediate container 3aa45abb4d67
 ---> dd4a7076b105
Step 4/6 : ADD VERSION /var/www/html/
 ---> f0cff03e7259
Step 5/6 : ADD Dockerfile.tar.gz /var/www/html/
 ---> 637b3b1540b6
Step 6/6 : RUN cat Dockerfile
 ---> Running in db3bd3b677bf
FROM basic-base

ARG GIT_COMMIT=unknown
LABEL git-commit=$GIT_COMMIT

CMD mkdir -p /var/www/html/

WORKDIR /var/www/html/

ADD Dockerfile.tar.gz /var/www/html/
RUN cat DockerfileRemoving intermediate container db3bd3b677bf
 ---> 75f435ebb18a
Successfully built 75f435ebb18a
Successfully tagged chomskim/basic-app:1.0.0

docker images
REPOSITORY           TAG         IMAGE ID       CREATED              SIZE
chomskim/basic-app   1.0.0       75f435ebb18a   About a minute ago   12.5MB

docker save -o /tmp/basic-app.tar chomskim/basic-app:1.0.0
du -sh /tmp/basic-app.tar
13M     /tmp/basic-app.tar

docker rmi 75f435ebb18a
docker load -i /tmp/basic-app.tar
e10e11b829d3: Loading layer [==================================================>]   2.56kB/2.56kB
d930774f6fd9: Loading layer [==================================================>]  3.584kB/3.584kB
0c64a939cb80: Loading layer [==================================================>]  3.584kB/3.584kB
Loaded image: chomskim/basic-app:1.0.0

docker images
REPOSITORY           TAG         IMAGE ID       CREATED          SIZE
chomskim/basic-app   1.0.0       75f435ebb18a   6 minutes ago    12.5MB

```

### Exercise 3.09: Storing Docker Images in Docker Hub and Deleting the Repository

```sh
docker tag basic-app chomskim/basic-app:ver1
docker images
REPOSITORY           TAG         IMAGE ID       CREATED          SIZE
chomskim/basic-app   1.0.0       75f435ebb18a   12 minutes ago   12.5MB
chomskim/basic-app   ver1        c60c31323b46   21 minutes ago   12.5MB
...

docker push chomskim/basic-app:ver1
The push refers to repository [docker.io/chomskim/basic-app]
e29fa1499ce8: Pushed
da07d3758ca2: Pushed
210a90ae4e07: Pushed
994393dc58e7: Mounted from library/alpine
ver1: digest: sha256:7d893d54c70246be6ddd84af03c37bd7b27a68c01a424056030aa09fdadbcff2 size: 1153

```

### Exercise 3.10: Creating a Local Docker Registry

```sh
[/etc/hosts]
docker pull registry
Using default tag: latest
latest: Pulling from library/registry
213ec9aee27d: Already exists
4583459ba037: Pull complete
6f6a6c5733af: Pull complete
b136d5c19b1d: Pull complete
fd4a5435f342: Pull complete
Digest: sha256:2e830e8b682d73a1b70cac4343a6a541a87d5271617841d87eeb67a824a5b3f2
Status: Downloaded newer image for registry:latest
docker.io/library/registry:latest

docker run -d -p 5000:5000 --restart=always --name registry registry

docker ps
CONTAINER ID   IMAGE      COMMAND                  CREATED          STATUS          PORTS                                       NAMES
32fdf4aa1513   registry   "/entrypoint.sh /etc…"   19 seconds ago   Up 18 seconds   0.0.0.0:5000->5000/tcp, :::5000->5000/tcp   registry

docker tag chomskim/basic-app:ver1 dev.docker.local:5000/basic-app:ver1

docker push dev.docker.local:5000/basic-app:ver1
The push refers to repository [dev.docker.local:5000/basic-app]
e29fa1499ce8: Pushed
da07d3758ca2: Pushed
210a90ae4e07: Pushed
994393dc58e7: Pushed
ver1: digest: sha256:7d893d54c70246be6ddd84af03c37bd7b27a68c01a424056030aa09fdadbcff2 size: 1153

docker rmi dev.docker.local:5000/basic-app:ver1

docker pull dev.docker.local:5000/basic-app:ver1
docker images
REPOSITORY                        TAG         IMAGE ID       CREATED             SIZE
chomskim/basic-app                1.0.0       75f435ebb18a   29 minutes ago      12.5MB
dev.docker.local:5000/basic-app   ver1        c60c31323b46   38 minutes ago      12.5MB
...

```
