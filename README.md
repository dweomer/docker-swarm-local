#   Docker Swarm Local
> This nifty little project is the direct result of a discussion with [@allingeek](https://twitter.com/allingeek) on the PhoenixDocker [#Meetup](http://www.meetup.com/Docker-Phoenix/)
> Slack channel. The composition presented here is a derivative of his work.

### UPDATE:
This project has been updated to work with Docker Swarm v2 aka Swarm Mode as introduced in 1.12.
Support for Swarm v1 has been relegated to the [`legacy`](https://github.com/dweomer/docker-swarm-local/tree/legacy) branch.

##  Have Docker, Will Swarm
Are you working on the Docker Engine COBOL client, aka the New Hotness &trade; and need a handy, throw-away Docker daemon/swarm to test against?

##  Getting Started
You will need to install [Docker Engine](https://github.com/docker/docker/releases) and [Docker Compose](https://github.com/docker/compose/releases).
This project was tested against:

* Engine 17.05 with Compose 1.13

If you run into any problems executing this composition first try installing the latest Docker Engine/Compose and/or
modifying the composition to match your local Docker Engine version.

### `docker-compose up -d && sleep 15`

Start up our swarm composition:

```
Creating network "dockerswarmlocal_swarm" with driver "bridge"
Creating volume "dockerswarmlocal_swarm" with default driver
Creating swarm-manager-1 ...
Creating swarm-manager-3 ...
Creating swarm-manager-2 ...
Creating swarm-manager-3
Creating swarm-manager-2
Creating swarm-manager-1 ... done
Creating swarm-manager-3-join ...
Creating swarm-manager-1-init ...
Creating swarm-manager-2 ... done
Creating swarm-manager-2-join
Creating swarm-manager-3-join
Creating swarm-manager-3-join ... done
```

### `docker ps -a`

Make sure there are no errors:

```
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                      PORTS                    NAMES
5ed1f8a4aa3c        docker:17.05        "docker-entrypoint..."   22 seconds ago      Exited (0) 13 seconds ago                            swarm-manager-1-init
3fb6728aa99c        docker:17.05        "docker-entrypoint..."   22 seconds ago      Exited (0) 3 seconds ago                             swarm-manager-3-join
db7049ff198f        docker:17.05        "docker-entrypoint..."   22 seconds ago      Exited (0) 5 seconds ago                             swarm-manager-2-join
b3f34e0f52ae        docker:17.05-dind   "dockerd-entrypoin..."   25 seconds ago      Up 22 seconds               0.0.0.0:2375->2375/tcp   swarm-manager-1
f7e235bf7951        docker:17.05-dind   "dockerd-entrypoin..."   25 seconds ago      Up 21 seconds               2375/tcp                 swarm-manager-3
1d8301c392c2        docker:17.05-dind   "dockerd-entrypoin..."   25 seconds ago      Up 22 seconds               2375/tcp                 swarm-manager-2
```

### `DOCKER_HOST=tcp://localhost:2375 docker info`

Witness our handiwork. The output has been edited to show only that the Swarm is active and has the expected number of managers.

```
# <snipped>
Swarm: active
 NodeID: pk6dt77l9fouonbv467g0xpwp
 Is Manager: true
 ClusterID: np11cwz99drfq7bswl4w0b796
 Managers: 3
 Nodes: 3
 Orchestration:
  Task History Retention Limit: 5
 Raft:
  Snapshot Interval: 10000
  Number of Old Snapshots to Retain: 0
  Heartbeat Tick: 1
  Election Tick: 3
 Dispatcher:
  Heartbeat Period: 5 seconds
 CA Configuration:
  Expiry Duration: 3 months
 Node Address: 172.19.0.3
 Manager Addresses:
  172.19.0.2:2377
  172.19.0.3:2377
  172.19.0.4:2377
# <snipped>
```

### `DOCKER_HOST=tcp://localhost:2375 docker service create --detach false --mode global --name nginx --publish 80:80 nginx`

Create a (global) service:

```
qw9sie41yjpjjc6zthzeth2kq
overall progress: 3 out of 3 tasks
pk6dt77l9fou: running   [==================================================>]
gb1cat9b5axy: running   [==================================================>]
9vcep8pze3az: running   [==================================================>]
verify: Waiting 1 seconds to verify that tasks are stable...
```

### `DOCKER_HOST=tcp://localhost:2375 docker service ps nginx`

Verify that the service is running:

```
ID                  NAME                              IMAGE               NODE                DESIRED STATE       CURRENT STATE           ERROR               PORTS
ms43yket9uan        nginx.9vcep8pze3az6kw8386jv9adi   nginx:latest        f7e235bf7951        Running             Running 3 minutes ago                       
ie9aftg7oqxq        nginx.gb1cat9b5axyielx1j7h4zkpc   nginx:latest        1d8301c392c2        Running             Running 3 minutes ago                       
3fgj7ejntjks        nginx.pk6dt77l9fouonbv467g0xpwp   nginx:latest        b3f34e0f52ae        Running             Running 3 minutes ago                       
```

### `DOCKER_HOST=tcp://172.19.0.2:2375 docker ps -a`

Verify that each node has one instance/task of the service.

```
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS               NAMES
264e04cc3489        nginx:latest        "nginx -g 'daemon ..."   About a minute ago   Up About a minute   80/tcp              nginx.gb1cat9b5axyielx1j7h4zkpc.ie9aftg7oqxqr2woltfi9car3
```

### `DOCKER_HOST=tcp://172.19.0.3:2375 docker ps -a`

Verify that each node has one instance/task of the service.

```
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS               NAMES
d1339e3da3d0        nginx:latest        "nginx -g 'daemon ..."   About a minute ago   Up About a minute   80/tcp              nginx.pk6dt77l9fouonbv467g0xpwp.3fgj7ejntjks1je3a4dbjd5az
```

### `DOCKER_HOST=tcp://172.19.0.4:2375 docker ps -a`

Verify that each node has one instance/task of the service.

```
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS               NAMES
6d0bde3739e0        nginx:latest        "nginx -g 'daemon ..."   About a minute ago   Up About a minute   80/tcp              nginx.9vcep8pze3az6kw8386jv9adi.ms43yket9uan4oksumra6l0dc
```

### `docker exec -it swarm-manager-1 sh -c 'curl -v http://$(hostname)'`

Does the service actually work? Let's cURL it! Because the service is global this command should work regardless if which manager container you target.

```
* Rebuilt URL to: http://b3f34e0f52ae/
*   Trying 172.19.0.3...
* TCP_NODELAY set
* Connected to b3f34e0f52ae (172.19.0.3) port 80 (#0)
> GET / HTTP/1.1
> Host: b3f34e0f52ae
> User-Agent: curl/7.52.1
> Accept: */*
>
< HTTP/1.1 200 OK
< Server: nginx/1.13.0
< Date: Fri, 19 May 2017 15:39:43 GMT
< Content-Type: text/html
< Content-Length: 612
< Last-Modified: Tue, 25 Apr 2017 11:30:31 GMT
< Connection: keep-alive
< ETag: "58ff3357-264"
< Accept-Ranges: bytes
<
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
* Curl_http_done: called premature == 0
* Connection #0 to host b3f34e0f52ae left intact
```

### `docker network inspect dockerswarmlocal_swarm`

The bridge network:

```
[
    {
        "Name": "dockerswarmlocal_swarm",
        "Id": "c9179e8cca413d69e7aa7beedd59ec21963590673c7946c93778566c407b8fc3",
        "Created": "2017-05-19T08:27:47.500165771-07:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.19.0.0/16",
                    "Gateway": "172.19.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": true,
        "Ingress": false,
        "Containers": {
            "1d8301c392c26d196a7690d8b448647b76b1ce3c3820ac4591e8e8954da76c7e": {
                "Name": "swarm-manager-2",
                "EndpointID": "b9f77ac42d44337b709390a7beed828668bf6a14974882e9ea628fbc88ffa985",
                "MacAddress": "02:42:ac:13:00:02",
                "IPv4Address": "172.19.0.2/16",
                "IPv6Address": ""
            },
            "b3f34e0f52aeed0f6293bebe23654c0a3e8edeedb0dd83628699188862153f7c": {
                "Name": "swarm-manager-1",
                "EndpointID": "1ab9c47448f25683b4d6ba79345b0e6009a5522c4907daf1c1170ae6ddf4b94d",
                "MacAddress": "02:42:ac:13:00:03",
                "IPv4Address": "172.19.0.3/16",
                "IPv6Address": ""
            },
            "f7e235bf7951364f0769b3092451a03b537e408be34f2a6dea46fc80c9330333": {
                "Name": "swarm-manager-3",
                "EndpointID": "22be2f21126275b4712404566b31123a7c3baa20fb6fa5584b662787088ee257",
                "MacAddress": "02:42:ac:13:00:04",
                "IPv4Address": "172.19.0.4/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {
            "com.docker.compose.network": "swarm",
            "com.docker.compose.project": "dockerswarmlocal"
        }
    }
]
```

Because the Swarm's backbone network is the host-local bridge network one can access the mesh endpoints directly, i.e.:

```
for h in $(DOCKER_HOST=tcp://localhost:2375 docker info 2>/dev/null | grep 2377 | sed -e 's/[:]2377//g'); do httping -c 1 http://$h; done
```

Should get you output that looks like:

```
PING 172.19.0.2:80 (/):
connected to 172.19.0.2:80 (237 bytes), seq=0 time=  0.62 ms
--- http://172.19.0.2/ ping statistics ---
1 connects, 1 ok, 0.00% failed, time 1001ms
round-trip min/avg/max = 0.6/0.6/0.6 ms
PING 172.19.0.3:80 (/):
connected to 172.19.0.3:80 (237 bytes), seq=0 time=  1.11 ms
--- http://172.19.0.3/ ping statistics ---
1 connects, 1 ok, 0.00% failed, time 1002ms
round-trip min/avg/max = 1.1/1.1/1.1 ms
PING 172.19.0.4:80 (/):
connected to 172.19.0.4:80 (237 bytes), seq=0 time=  0.82 ms
--- http://172.19.0.4/ ping statistics ---
1 connects, 1 ok, 0.00% failed, time 1001ms
round-trip min/avg/max = 0.8/0.8/0.8 ms
```

##  Cleaning Up
### `docker-compose down -v`

##  Copyright Notice
>The [MIT License](LICENSE) ([MIT](https://opensource.org/licenses/MIT))
>
> Copyright &copy; 2016-2017 [Jacob Blain Christen](https://github.com/dweomer) and [Jeff Nickoloff - All in Geek Consulting Services, LLC](https://github.com/allingeek)
>
> Permission is hereby granted, free of charge, to any person obtaining a copy of
> this software and associated documentation files (the "Software"), to deal in
> the Software without restriction, including without limitation the rights to
> use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of
> the Software, and to permit persons to whom the Software is furnished to do so,
> subject to the following conditions:
>
> The above copyright notice and this permission notice shall be included in all
> copies or substantial portions of the Software.
>
> THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
> IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS
> FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR
> COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER
> IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN
> CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
