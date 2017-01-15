#   Docker Swarm Local
> This nifty little project is the direct result of a discussion with [@allingeek](https://twitter.com/allingeek) on the PhoenixDocker [#Meetup](http://www.meetup.com/Docker-Phoenix/)
> Slack channel. The composition presented here is a derivative of his work.

##  Have Docker, Will Swarm
Are you working on the Docker Engine COBOL client, aka the New Hotness &trade; and need a handy, throw-away Docker daemon/swarm to test against?
Sure, you could setup something using [Docker Machine, possibly even leveraging Consul, on say, Digital Ocean](https://github.com/dweomer/docker-swarm-consul)
but ideally you could leverage something quick and local, amirite?

Still With me? Yeah, so, this fantastic little [docker-compose.yml](docker-compose.yml) ([v1](docker-compose.v1.yml) or [v2](docker-compose.v2.yml))
will fire up a Swarm of Docker Daemons running on your local (read any) Docker Host with all communications on the bridge network.

##  Getting Started
You will need to install [Docker Engine](https://github.com/docker/docker/releases) and [Docker Compose](https://github.com/docker/compose/releases).
This project was tested against:

* Engine 1.10 with Compose 1.6
* Engine 1.11 with Compose 1.6
* Engine 1.11 with Compose 1.7

If you run into any problems executing this composition first try installing the latest Docker Engine/Compose and/or
modifying the composition to match your local Docker Engine version.

### `docker-compose up -d`

```
Creating dockerswarmlocal_sn1_1
Creating dockerswarmlocal_sn3_1
Creating dockerswarmlocal_sn2_1
Creating dockerswarmlocal_sn1lc_1
Creating dockerswarmlocal_sn3lc_1
Creating dockerswarmlocal_sn2lc_1
Creating dockerswarmlocal_sm_1
Creating dockerswarmlocal_smlc_1
```

### `docker ps -a`

```
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                     PORTS                   NAMES
7df1943de3d6        swarm               "/swarm manage -H tcp"   10 seconds ago      Up 6 seconds               0.0.0.0:3375->2375/tcp  dockerswarmlocal_sm_1
753f7a198293        docker              "docker-entrypoint.sh"   10 seconds ago      Exited (0) 2 seconds ago                           dockerswarmlocal_sn3lc_1
97cba0740412        docker              "docker-entrypoint.sh"   10 seconds ago      Exited (0) 3 seconds ago                           dockerswarmlocal_sn2lc_1
b41df9838d8b        docker              "docker-entrypoint.sh"   10 seconds ago      Exited (0) 1 seconds ago                           dockerswarmlocal_sn1lc_1
c4b0483f6aa1        docker:dind         "dockerd-entrypoint.s"   11 seconds ago      Up 9 seconds               2375/tcp                dockerswarmlocal_sn3_1
2bcc06bd7175        docker:dind         "dockerd-entrypoint.s"   11 seconds ago      Up 9 seconds               2375/tcp                dockerswarmlocal_sn1_1
7b8b04f2020d        docker:dind         "dockerd-entrypoint.s"   11 seconds ago      Up 9 seconds               2375/tcp                dockerswarmlocal_sn2_1
```

### `DOCKER_HOST=tcp://localhost:3375 docker info`

```
Containers: 4
 Running: 4
 Paused: 0
 Stopped: 0
Images: 3
Server Version: swarm/1.2.2
Role: primary
Strategy: spread
Filters: health, port, containerslots, dependency, affinity, constraint
Nodes: 3
 node-1: sn1:2375
  └ ID: EOXC:LGHA:DVVL:KLJN:HARD:3U7W:T6GG:S745:4EUZ:CFHG:TO4X:6ABX
  └ Status: Healthy
  └ Containers: 1
  └ Reserved CPUs: 0 / 8
  └ Reserved Memory: 0 B / 32.95 GiB
  └ Labels: executiondriver=, kernelversion=4.2.0-35-generic, operatingsystem=Alpine Linux v3.3 (containerized), storagedriver=vfs
  └ Error: (none)
  └ UpdatedAt: 2016-05-11T13:27:45Z
  └ ServerVersion: 1.11.1
 node-2: sn2:2375
  └ ID: C34E:SKZZ:MHC4:Y3Q2:DVMX:IEM6:WH3R:LWTF:BOAL:RIBE:Z3N7:I2IB
  └ Status: Healthy
  └ Containers: 1
  └ Reserved CPUs: 0 / 8
  └ Reserved Memory: 0 B / 32.95 GiB
  └ Labels: executiondriver=, kernelversion=4.2.0-35-generic, operatingsystem=Alpine Linux v3.3 (containerized), storagedriver=vfs
  └ Error: (none)
  └ UpdatedAt: 2016-05-11T13:28:11Z
  └ ServerVersion: 1.11.1
 node-3: sn3:2375
  └ ID: XSLU:UT4U:J5IQ:V3FP:BT6K:IOWB:SZR3:ZNGA:BECQ:25KP:XJLG:UYHE
  └ Status: Healthy
  └ Containers: 2
  └ Reserved CPUs: 0 / 8
  └ Reserved Memory: 0 B / 32.95 GiB
  └ Labels: executiondriver=, kernelversion=4.2.0-35-generic, operatingsystem=Alpine Linux v3.3 (containerized), storagedriver=vfs
  └ Error: (none)
  └ UpdatedAt: 2016-05-11T13:28:04Z
  └ ServerVersion: 1.11.1
Plugins:
 Volume:
 Network:
Kernel Version: 4.2.0-35-generic
Operating System: linux
Architecture: amd64
CPUs: 24
Total Memory: 98.85 GiB
Name: 7df1943de3d6
Docker Root Dir:
Debug mode (client): false
Debug mode (server): false
WARNING: No kernel memory limit support
```

### `DOCKER_HOST=tcp://localhost:3375 docker ps -a`

```
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS               NAMES
117872fbeb79        alpine              "sh -c 'echo NODE1; s"   About a minute ago   Up About a minute                       node-1/agitated_bhaskara
d8fc21857f7d        alpine              "sh -c 'echo SWARM; s"   About a minute ago   Up About a minute                       node-3/sharp_jang
92c9952fef2f        alpine              "sh -c 'echo NODE3; s"   About a minute ago   Up About a minute                       node-3/cranky_cori
b09d89c55cc5        alpine              "sh -c 'echo NODE2; s"   About a minute ago   Up About a minute                       node-2/sad_yonath
```

### `DOCKER_HOST=tcp://localhost:3375 docker ps -a --filter 'label=scheduled-by=sm`

```
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
d8fc21857f7d        alpine              "sh -c 'echo SWARM; s"   2 minutes ago       Up 2 minutes                            node-3/sharp_jang
```

### `DOCKER_HOST=tcp://localhost:3375 docker ps -a --format 'name={{.Names}}, launched-by={{.Label "launched-by"}}, scheduled-by={{.Label "scheduled-by"}}'`

```
name=node-1/agitated_bhaskara, launched-by=sn1lc, scheduled-by=sn1
name=node-3/sharp_jang, launched-by=smlc, scheduled-by=sm
name=node-3/cranky_cori, launched-by=sn3lc, scheduled-by=sn3
name=node-2/sad_yonath, launched-by=sn2lc, scheduled-by=sn2
```

### `docker-compose scale smlc=2`

```
Starting dockerswarmlocal_smlc_1 ... done
Creating and starting dockerswarmlocal_smlc_2 ... done
```

### `DOCKER_HOST=tcp://localhost:3375 docker ps -a`

```
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS               NAMES
a3aea02187c8        alpine              "sh -c 'echo SWARM; s"   About a minute ago   Up About a minute                       node-2/clever_jepsen
60422b9d3456        alpine              "sh -c 'echo SWARM; s"   About a minute ago   Up About a minute                       node-1/sad_lovelace
117872fbeb79        alpine              "sh -c 'echo NODE1; s"   7 minutes ago        Up 7 minutes                            node-1/agitated_bhaskara
d8fc21857f7d        alpine              "sh -c 'echo SWARM; s"   7 minutes ago        Up 7 minutes                            node-3/sharp_jang
92c9952fef2f        alpine              "sh -c 'echo NODE3; s"   7 minutes ago        Up 7 minutes                            node-3/cranky_cori
b09d89c55cc5        alpine              "sh -c 'echo NODE2; s"   7 minutes ago        Up 7 minutes                            node-2/sad_yonath
```

### `DOCKER_HOST=tcp://localhost:3375 docker ps -a --filter 'label=scheduled-by=sm`

```
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS               NAMES
a3aea02187c8        alpine              "sh -c 'echo SWARM; s"   About a minute ago   Up About a minute                       node-2/clever_jepsen
60422b9d3456        alpine              "sh -c 'echo SWARM; s"   About a minute ago   Up About a minute                       node-1/sad_lovelace
d8fc21857f7d        alpine              "sh -c 'echo SWARM; s"   6 minutes ago        Up 6 minutes                            node-3/sharp_jang
```

### `DOCKER_HOST=tcp://localhost:3375 docker ps -a --format 'name={{.Names}}, launched-by={{.Label "launched-by"}}, scheduled-by={{.Label "scheduled-by"}}'`

```
name=node-2/clever_jepsen, launched-by=smlc, scheduled-by=sm
name=node-1/sad_lovelace, launched-by=smlc, scheduled-by=sm
name=node-1/agitated_bhaskara, launched-by=sn1lc, scheduled-by=sn1
name=node-3/sharp_jang, launched-by=smlc, scheduled-by=sm
name=node-3/cranky_cori, launched-by=sn3lc, scheduled-by=sn3
name=node-2/sad_yonath, launched-by=sn2lc, scheduled-by=sn2
```

##  Cleaning Up
### `docker-compose down -v`

##  Copyright Notice
>The [MIT License](LICENSE) ([MIT](https://opensource.org/licenses/MIT))
>
> Copyright &copy; 2016 [Jacob Blain Christen](https://github.com/dweomer) and [Jeff Nickoloff - All in Geek Consulting Services, LLC](https://github.com/allingeek)
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
