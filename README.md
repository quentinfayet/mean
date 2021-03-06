# My MEAN stack under Docker

I've decided to Dockerize the MEAN stack, in order to ease my futur developments.

This stack is really basic, i really welcome improvments.

There is two ways to set up this stack. The first one is by getting the images from my DockerHub repository ; the second one is by building the images on your own, then runing it.

## Getting from the DockerHub

First of all, this is a quick way to set up the environnement. If you want to know more about the different images, please go to the "Building & Running on your own" section down bellow.

### Running the Datastore container

```shell
$> docker run -d --name my_datastore quentinfayet/datastore:0.1
```

### Running the Mongo container 

```shell
$> docker run -d --volumes-from my_datastore --name my_mongodb -p 27017:27017 quentinfayet/mongodb:0.1
```

### Running the Node.js container

```shell
$> docker run --name my_node -d -i -t  -v /data/app:/data/app -p 8888:8888 quentinfayet/node:0.1
```

## Building & Running on your own

### The DataStore

What I call the "DataStore" is the container I use to store persistent data. As Docker creates a volume for each container on the host filesystem, I created this container to share its volume with other containers (like a hub).

You can build it by running a simple build command inside the directory.

(Assuming you're building with the tag "datastore") :

```shell
$> cd datastore
$> docker build -t datastore .
```

Then, you need to run it at least one time for the volume to be created on the host filesystem (I gave it the name of "my_datastore") :

```shell
$> docker run -d --name my_datastore datastore
```
You can check its existence with the following docker command : 

            
```shell
$> docker ps -a
```

The volume has been created on the host filesystem, you can check it by listing the files in the /var/lib/docker/volumes directory.

### The Mongo

The MongoDB container can be built by running the build command in the appropriate directory (I gave him the tag "mongo"):

```shell
$> cd mongo 
$> docker build -t mongo .
```

The "tricky" part is that I don't want the Mongo container to store data on his own. I would prefer that it stores data in the appropriate "datastore" container we built earlier.

In order to do so, we can use the --volumes-from parameter of the Docker "run" command. I also want to forward the default mongo port (27017) to the host port 27017. Finally, i run it in detached mode.

```shell
$> docker run -d --volumes-from my_datastore --name my_mongodb -p 27017:27017 mongo
```

Now, you can try to connect the host on the 27017 port, that will be forwarded to the container's 27017 port.

### The Node.js

The final part of the BackOffice of the MEAN Stack is the Node.js server, along with Express framework.

Building it :

```shell
$> cd node-express 
$> docker build -t node .
```

In my stack, I assumed the app directory would be the /data/app directory on the host filesystem. I defined it in the package.json file. The base image I used in this container is the official node image. This image assumes that you have a package.json file in your directory, with a start script defined in it.

Then, i run the container in detached mode, and sharing the host's directory /data/app and port-forwarding the 8888 port.

```shell
$> docker run --name my_node -d -i -t  -v /data/app:/data/app -p 8888:8888 node
```
