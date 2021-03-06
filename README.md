# MesosClusterSpyke

Vagrantfile and some shell scripts for createing Mesos + Marathon + Chronos + Docker cluster

If you just try Mesos cluster, it will be painful. This set of files makes you easy to generate a cluster environment of Mesos.
You can refer [This article written in Japanese](http://qiita.com/TsuyoshiUshio@github/items/9d8f5b952b635d94ae6f) if you like.

## Getting started

### 1. Clone and installation

Prerequisite

```
Mac OS X Yosemite version 10.10.2 
Virtual Box 4.3.26
vagrant 1.7.2
git 2.2.1
```

I try it using Windows 8.1 and vagrant 1.7.2. it works!

Just clone it then vagrant up and reload. It will take about 40 minutes.

```
$ git clone https://github.com/TsuyoshiUshio/MesosClusterSpyke.git
$ cd MesosClusterSpyke
```

If you want to change Timezone and Localize it, prease edit [common_ubuntu.sh](https://github.com/TsuyoshiUshio/MesosClusterSpyke/blob/master/common_ubuntu.sh).


```
$ vagrant up
$ vagrant reload
```

### 2. Use it via Browser

![Cluster structure](https://qiita-image-store.s3.amazonaws.com/0/3470/1033cf45-84bc-2560-5c67-7933e6a41e83.jpeg)

```
Apache Mesos :   http://192.168.33.10:5050
Marathon     :   http://192.168.33.10:8080
Chronos      :   http://192.168.33.10:8081
```

**NOTE:Use leader node for Marathon and Chronos**

Chronos and Marathon work only on learder node of Mesos. You may need to stop Chronos and Marathon which is working on non leader nodes.

To get to know the leader node, Open Apache Mesos pages (e.g. http://192.168.33.10:5050) then it will automatically redirect to the leader node.

### 3. How to stop the non leader nodes

*Stop Chronos*

```
$ vagrant ssh master_01

vagrant@vagrant-ubuntu-trusty-64:~$ sudo initctl stop chronos
```

*Stop Marathon*

```
$ vagrant ssh master_01

vagrant@vagrant-ubuntu-trusty-64:~$ sudo initctl stop marathon
```

![Screen Shot 2015-05-12 at 21.57.54.png](https://qiita-image-store.s3.amazonaws.com/0/3470/56452231-cbe0-793d-697e-0e0fc1f54d0d.png)
![Screen Shot 2015-05-12 at 21.58.01.png](https://qiita-image-store.s3.amazonaws.com/0/3470/cd373de1-e890-2038-0b21-34aca7b356d8.png)
![Screen Shot 2015-05-12 at 22.57.43.png](https://qiita-image-store.s3.amazonaws.com/0/3470/854306ff-786f-80c2-457e-07d866076cd8.png)

### 4. Deploy your Docker container using Marathon

Create Docker.json file like this.

```
{
	"container": {
			"type": "DOCKER",
					"docker": {
						"image": "tsuyoshiushio/aservice",
						"network": "BRIDGE",
						"portMappings": [
							{ "containerPort": 80, "hostPort": 0, "servicePort": 0, "protocol": "tcp"}
						]
					}
	},
	"id": "aservice",
	"instances": 1,
	"cpus": 0.5,
	"mem": 512,
	"uris": []
}
```

Then execute this command. You need to change the ipaddress(192.168.33.10) into others(192.168.33.11, 192.168.33.12) if it is not a master node.

```
$ curl -X POST -H "Content-Type: application/json" http://192.168.33.10:8080/v2/apps -d@Docker.json
```

Marathon automatically allocate a port for this application. HostPort/ServierPort = 0 meanins allocate it automatically. You can find the port on Marathon at app>aservice Page. Click the link then you can see a demo webpage of a docker container!

Happy clustering!
