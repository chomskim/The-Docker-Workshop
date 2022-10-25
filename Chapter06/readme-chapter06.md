## Chapter 06

### Exercise 6.01: Hands-On with Docker Networking

```sh
docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
29f585e1bc70   bridge    bridge    local
0d02a71e8915   host      host      local
13dc05c628c5   none      null      local

ifconfig
docker0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255
        inet6 fe80::42:52ff:fe89:1fb4  prefixlen 64  scopeid 0x20<link>
        ether 02:42:52:89:1f:b4  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 38  bytes 4826 (4.8 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

sudo aa-remove-unknown[when can not stop some container item]
docker rmi -f $(docker images -a -q)

docker run -d --name webserver1 nginx:latest
docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS     NAMES
a24b81e32f0b   nginx:latest   "/docker-entrypoint.…"   18 seconds ago   Up 15 seconds   80/tcp    webserver1

docker inspect webserver1
...
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "29f585e1bc70bf5ff0d0ab1993c647499e02d51cca2ade10e3b0f12918e1980c",
                    "EndpointID": "8818cc92278178a50224ea90f9607def0e8e0132b4abf5aeaa29cc29a9e05956",
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.2",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:11:00:02",
                    "DriverOpts": null
                }
            }
...
[localhost Chrome]
http://172.17.0.2/ --> Welcome to nginx!

docker run -d -p 8080:80 --name webserver2 nginx:latest
docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS                                   NAMES
c064072afd41   nginx:latest   "/docker-entrypoint.…"   26 seconds ago   Up 25 seconds   0.0.0.0:8080->80/tcp, :::8080->80/tcp   webserver2
a24b81e32f0b   nginx:latest   "/docker-entrypoint.…"   6 minutes ago    Up 6 minutes    80/tcp                                  webserver1

[localhost Chrome]
http://localhost:8080/ --> Welcome to nginx!

docker inspect webserver2
...
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "29f585e1bc70bf5ff0d0ab1993c647499e02d51cca2ade10e3b0f12918e1980c",
                    "EndpointID": "c070cf5aa0c912e13fe06b59343b6529efe6c9a825de30ac364aa3634d396c21",
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.3",
...

docker exec -it webserver1 /bin/bash
root@a24b81e32f0b:/# apt-get update && apt-get install -y inetutils-ping
root@a24b81e32f0b:/# ping 172.17.0.3
PING 172.17.0.3 (172.17.0.3): 56 data bytes
64 bytes from 172.17.0.3: icmp_seq=0 ttl=64 time=0.143 ms
64 bytes from 172.17.0.3: icmp_seq=1 ttl=64 time=0.097 ms
64 bytes from 172.17.0.3: icmp_seq=2 ttl=64 time=0.098 ms
...
root@a24b81e32f0b:/# apt-get install -y curl
curl 172.17.0.3
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...

```

### Exercise 6.02: Working with Docker DNS

```sh
docker run -itd --name containerlink1 alpine:latest
docker run -itd --name containerlink2 --link containerlink1 alpine:latest
docker exec -it containerlink2 /bin/sh
/ # ping containerlink1
PING containerlink1 (172.17.0.2): 56 data bytes
64 bytes from 172.17.0.2: seq=0 ttl=64 time=0.433 ms
64 bytes from 172.17.0.2: seq=1 ttl=64 time=0.104 ms


docker exec -it containerlink1 /bin/sh
/ # ping containerlink2
ping: bad address 'containerlink2'

docker network create dnsnet1 --subnet 192.168.71.0/24 --gateway 192.168.71.1
docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
29f585e1bc70   bridge    bridge    local
2ead5bb7f4b4   dnsnet1   bridge    local
0d02a71e8915   host      host      local
13dc05c628c5   none      null      local

docker network inspect dnsnet1
...
            "Config": [
                {
                    "Subnet": "192.168.71.0/24",
                    "Gateway": "192.168.71.1"
                }
            ]
...

ifconfig
br-2ead5bb7f4b4: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 192.168.71.1  netmask 255.255.255.0  broadcast 192.168.71.255
        ether 02:42:d5:14:9f:cb  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

docker0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255
        inet6 fe80::42:52ff:fe89:1fb4  prefixlen 64  scopeid 0x20<link>
        ether 02:42:52:89:1f:b4  txqueuelen 0  (Ethernet)
        RX packets 1384  bytes 78919 (78.9 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 2662  bytes 9061862 (9.0 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

docker run -itd --network dnsnet1 --network-alias alpinedns1 --name alpinedns1 alpine:latest
8538b427620e33270399c510974ca5fcf86341951b67d7b5cf308c187b905943

docker run -itd --network dnsnet1 --network-alias alpinedns2 --name alpinedns2 alpine:latest
9fdd7db3e329f27556f867b23f7a8d6ee8384f944ace691ea268abb28ca73510

docker ps
CONTAINER ID   IMAGE           COMMAND     CREATED          STATUS          PORTS     NAMES
9fdd7db3e329   alpine:latest   "/bin/sh"   21 seconds ago   Up 19 seconds             alpinedns2
8538b427620e   alpine:latest   "/bin/sh"   2 minutes ago    Up 2 minutes              alpinedns1
7bafcc6cdd47   alpine:latest   "/bin/sh"   13 minutes ago   Up 13 minutes             containerlink2
007a503c7621   alpine:latest   "/bin/sh"   14 minutes ago   Up 14 minutes             containerlink1

docker inspect alpinedns1
...
            "Networks": {
                "dnsnet1": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": [
                        "alpinedns1",
                        "8538b427620e"
                    ],
                    "NetworkID": "2ead5bb7f4b4484e05d0113a4514305c1c6be861ecdec703c39147f5e9ca15d7",
                    "EndpointID": "8775eda4459fe971db58887ad2873c1ad0a9c85e58f0b89e7db19e12258b44e4",
                    "Gateway": "192.168.71.1",
                    "IPAddress": "192.168.71.2",
                    "IPPrefixLen": 24,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:c0:a8:47:02",
                    "DriverOpts": null
                }
            }
...
docker inspect alpinedns2
...
            "Networks": {
                "dnsnet1": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": [
                        "alpinedns2",
                        "9fdd7db3e329"
                    ],
                    "NetworkID": "2ead5bb7f4b4484e05d0113a4514305c1c6be861ecdec703c39147f5e9ca15d7",
                    "EndpointID": "6a1c7b8d0365c9ddb9eaa4af85f565ce9be7d82a15dea15c7bf0464fddec1bd5",
                    "Gateway": "192.168.71.1",
                    "IPAddress": "192.168.71.3",
                    "IPPrefixLen": 24,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:c0:a8:47:03",
                    "DriverOpts": null
                }
            }
...
docker exec -it alpinedns1 /bin/sh
/ # ping alpinedns2
PING alpinedns2 (192.168.71.3): 56 data bytes
64 bytes from 192.168.71.3: seq=0 ttl=64 time=1.006 ms
64 bytes from 192.168.71.3: seq=1 ttl=64 time=0.161 ms
64 bytes from 192.168.71.3: seq=2 ttl=64 time=0.109 ms

/ # cat /etc/hosts
127.0.0.1       localhost
::1     localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
192.168.71.2    8538b427620e

docker exec -it alpinedns2 /bin/sh
/ # ping alpinedns1
PING alpinedns1 (192.168.71.2): 56 data bytes
64 bytes from 192.168.71.2: seq=0 ttl=64 time=0.224 ms
64 bytes from 192.168.71.2: seq=1 ttl=64 time=0.107 ms
64 bytes from 192.168.71.2: seq=2 ttl=64 time=0.109 ms

/ # cat /etc/hosts
127.0.0.1       localhost
::1     localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
192.168.71.3    9fdd7db3e329

docker stop alpinedns2 alpinedns1 containerlink2 containerlink1
docker system prune -fa

```
