## Chapter 02 Command

### Exercise 2.02: Creating Our First Docker Image

```sh
mkdir custom-docker-image
cp Dockerfile custom-docker-image
cd custom-docker-image
docker image build -t welcome:1.0 .
docker image build -t welcome:2.0 .
docker image list
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
welcome      1.0       b9be84fa8362   5 minutes ago   77.8MB
welcome      2.0       b9be84fa8362   5 minutes ago   77.8MB
ubuntu       latest    216c552ea5ba   2 weeks ago     77.8MB

docker run welcome:1.0
You are reading The Docker Workshop

docker run welcome:1.0 "Docker Beginner's Guide"
You are reading Docker Beginner's Guide
```

### Exercise 2.03: Using ENV and ARG Directives in a Dockerfile

```sh
cd Exercise2.03
mkdir env-arg-exercise
cp Dockerfile env-arg-exercise
cd env-arg-exercise
docker image build -t env-arg --build-arg TAG=19.04 .
docker image list
REPOSITORY   TAG       IMAGE ID       CREATED              SIZE
env-arg      latest    f2abefca3269   About a minute ago   70MB
welcome      1.0       b9be84fa8362   18 minutes ago       77.8MB
welcome      2.0       b9be84fa8362   18 minutes ago       77.8MB
ubuntu       latest    216c552ea5ba   2 weeks ago          77.8MB
ubuntu       19.04     c88ac1f841b7   2 years ago          70MB

docker run env-arg
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=ee023d2b24d3
PUBLISHER=packt
APP_DIR=/usr/local/app/bin
HOME=/root
```

### Exercise 2.04: Using the WORKDIR, COPY, and ADD Directives in the Dockerfile

```sh
mkdir workdir-copy-add-exercise
cp Dockerfile workdir-copy-add-exercise
cp index.html workdir-copy-add-exercise

cd workdir-copy-add-exercise
docker system prune -af

docker image build -t workdir-copy-add .
...
Enabling conf security.
Enabling conf serve-cgi-bin.
Enabling site 000-default.
invoke-rc.d: could not determine current runlevel
invoke-rc.d: policy-rc.d denied execution of start.
Processing triggers for libc-bin (2.35-0ubuntu3.1) ...
Processing triggers for ca-certificates (20211016) ...
Updating certificates in /etc/ssl/certs...
0 added, 0 removed; done.
Running hooks in /etc/ca-certificates/update.d...
done.
Removing intermediate container 06aced265c53
 ---> b25f9a707ad0
Step 3/6 : WORKDIR /var/www/html/
 ---> Running in 3fb436c82eda
Removing intermediate container 3fb436c82eda
 ---> c8fc81d0f682
Step 4/6 : COPY index.html .
 ---> da0bca71bf56
Step 5/6 : ADD https://seeklogo.com/images/M/moby-logo-EB8A3B8E0C-seeklogo.com.png?v=637792398800000000 ./logo.png
Downloading [==================================================>]   34.1kB/34.1kB

 ---> da06b473edf5
Step 6/6 : CMD ["ls"]
 ---> Running in fb07fb4e7a9f
Removing intermediate container fb07fb4e7a9f
 ---> ce5c411698c1
Successfully built ce5c411698c1
Successfully tagged workdir-copy-add:latest

docker run workdir-copy-add
index.html
logo.png

```

### Exercise 2.05: Using USER Directive in the Dockerfile

```sh
docker image build -t user .

docker run user
www-data
```

### Exercise 2.06: Using VOLUME Directive in the Dockerfile

```sh
docker image build -t volume .

docker run --interactive --tty --name volume-container volume /bin/bash
root@2e2a5ca5d8db:/# cd /var/log/apache2/
root@2e2a5ca5d8db:/var/log/apache2# ls -l
total 0
-rw-r----- 1 root adm 0 Oct 21 14:24 access.log
-rw-r----- 1 root adm 0 Oct 21 14:24 error.log
-rw-r----- 1 root adm 0 Oct 21 14:24 other_vhosts_access.log
root@2e2a5ca5d8db:/var/log/apache2# exit
exit

docker container inspect volume-container
...
        "Mounts": [
            {
                "Type": "volume",
                "Name": "a8e78932713a4f0bc5403c516b2048c5a269c6375ffdbee0729bdc426936fac4",
                "Source": "/var/snap/docker/common/var-lib-docker/volumes/a8e78932713a4f0bc5403c516b2048c5a269c6375ffdbee0729bdc426936fac4/_data",
                "Destination": "/var/log/apache2",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            }
        ],
...
docker volume inspect a8e78932713a4f0bc5403c516b2048c5a269c6375ffdbee0729bdc426936fac4
[
    {
        "CreatedAt": "2022-10-22T09:16:48+09:00",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/snap/docker/common/var-lib-docker/volumes/a8e78932713a4f0bc5403c516b2048c5a269c6375ffdbee0729bdc426936fac4/_data",
        "Name": "a8e78932713a4f0bc5403c516b2048c5a269c6375ffdbee0729bdc426936fac4",
        "Options": null,
        "Scope": "local"
    }
]
sudo ls -l /var/snap/docker/common/var-lib-docker/volumes/a8e78932713a4f0bc5403c516b2048c5a269c6375ffdbee0729bdc426936fac4/_data
total 0
-rw-r----- 1 root adm 0 10월 21 23:24 access.log
-rw-r----- 1 root adm 0 10월 21 23:24 error.log
-rw-r----- 1 root adm 0 10월 21 23:24 other_vhosts_access.log

```

### Exercise 2.07: Using EXPOSE and HEALTHCHECK Directives in the Dockerfile

```sh
docker image build -t expose-healthcheck .
docker container run -p 80:80 --name expose-healthcheck-container -d expose-healthcheck
f6b5b61a9a59285d9c1aad6be23f98ba38c27a50c939bf17a187a689c81f41b5

docker container list
CONTAINER ID   IMAGE                COMMAND                  CREATED          STATUS                             PORTS                               NAMES
f6b5b61a9a59   expose-healthcheck   "apache2ctl -D FOREG…"   26 seconds ago   Up 25 seconds (health: starting)   0.0.0.0:80->80/tcp, :::80->80/tcp   expose-healthcheck-container

[Chrome]http://127.0.0.1/
docker container stop expose-healthcheck-container
Error response from daemon: cannot stop container: expose-healthcheck-container: permission denied

sudo aa-status
apparmor module is loaded.
63 profiles are loaded.
59 profiles are in enforce mode.
...
14 processes have profiles defined.
14 processes are in enforce mode.
   /usr/sbin/cups-browsed (21465)
   /usr/sbin/cupsd (21464)
   /usr/lib/cups/notifier/dbus (21481) /usr/sbin/cupsd
   /usr/lib/cups/notifier/dbus (21482) /usr/sbin/cupsd
   /usr/bin/dash (30261) docker-default
   /usr/sbin/apache2 (30307) docker-default
   /usr/sbin/apache2 (30308) docker-default
   /usr/sbin/apache2 (30309) docker-default
   /snap/docker/2285/bin/dockerd (740) snap.docker.dockerd
   /snap/docker/2285/bin/containerd (1837) snap.docker.dockerd
   /snap/docker/2285/bin/docker-proxy (30218) snap.docker.dockerd
   /snap/docker/2285/bin/docker-proxy (30224) snap.docker.dockerd
   /snap/docker/2285/bin/containerd-shim-runc-v2 (30240) snap.docker.dockerd
   /snap/snap-store/599/usr/bin/snap-store (2969) snap.snap-store.ubuntu-software
0 processes are in complain mode.
0 processes are unconfined but have a profile defined.

sudo aa-remove-unknown
Skipping profile in /etc/apparmor.d/disable: usr.bin.firefox
Skipping profile in /etc/apparmor.d/disable: usr.sbin.rsyslogd
Removing 'snap.snap-store.ubuntu-software-local-file'
Removing 'snap.snap-store.ubuntu-software'
...
Removing '/snap/core/13886/usr/lib/snapd/snap-confine//mount-namespace-capture-helper'
Removing '/snap/core/13886/usr/lib/snapd/snap-confine'

sudo aa-status
apparmor module is loaded.
28 profiles are loaded.
26 profiles are in enforce mode.
...
4 processes have profiles defined.
4 processes are in enforce mode.
   /usr/sbin/cups-browsed (21465)
   /usr/sbin/cupsd (21464)
   /usr/lib/cups/notifier/dbus (21481) /usr/sbin/cupsd
   /usr/lib/cups/notifier/dbus (21482) /usr/sbin/cupsd
0 processes are in complain mode.
0 processes are unconfined but have a profile defined.

docker container stop expose-healthcheck-container
expose-healthcheck-container[Success]

docker container ls -a
CONTAINER ID   IMAGE                COMMAND                  CREATED             STATUS                         PORTS     NAMES
f6b5b61a9a59   expose-healthcheck   "apache2ctl -D FOREG…"   26 minutes ago      Exited (137) 13 minutes ago              expose-healthcheck-container

docker container rm expose-healthcheck-container
expose-healthcheck-container
```

### Exercise 2.08: Using ONBUILD Directive in the Dockerfile

```sh
[onbuild-parent]
docker image build -t onbuild-parent .
docker container run -p 80:80 --name onbuild-parent-container -d onbuild-parent
[Chrome]http://127.0.0.1

docker container stop onbuild-parent-container
docker container rm onbuild-parent-container

[onbuild-child]
docker image build -t onbuild-child .
docker container run -p 80:80 --name onbuild-child-container -d onbuild-child
[Chrome]http://127.0.0.1
Learning Docker ONBUILD directive

docker container stop onbuild-child-container
docker container rm onbuild-child-container
```

### Activity 2.01: Running a PHP Application on a Docker Container

```sh
docker image build -t activity2.01 .
docker images
REPOSITORY       TAG       IMAGE ID       CREATED          SIZE
activity2.01     latest    836eedb52ccb   56 seconds ago   231MB

docker container run -p 80:80 --name activity2.01-container -d activity2.01
9bc42a452556df8dffba80b180791cd6cb23589e6dc640dbcdc14ecda6a49157

[Chrome]http://127.0.0.1/welcome.php
Good Morning

docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS                   PORTS                               NAMES
9bc42a452556   activity2.01   "apache2ctl -D FOREG…"   2 minutes ago   Up 2 minutes (healthy)   0.0.0.0:80->80/tcp, :::80->80/tcp   activity2.01-container

docker container stop activity2.01-container
docker container rm activity2.01-container

docker system prune -af
```
