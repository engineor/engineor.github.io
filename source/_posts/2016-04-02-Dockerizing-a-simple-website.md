---
title: Dockerizing a simple website
tags:
    - docker
    - static website
    - server
    - deployment
categories:
    - docker
    - tutorial

---
First tutorial of this blog, this has been our initial step in getting to deploy websites with Docker.

![Docker logo](https://static-lab.engineor.com/Dockerizing-a-simple-website/docker.png)

### Introduction
If you do not know Docker, please have a look at [their website](https://www.docker.com/what-docker) and the numerous presentation of the basic concepts. To keep it short, Docker is a container management system, allowing to have small contained processes instead of virtualising systems, therefore reducing the overhead introduced in the virtualisation.


![Docker: containers vs vms](https://static-lab.engineor.com/Dockerizing-a-simple-website/containers-vs-vms.png)
Retrieved from [Blog Logentries: an all inclusive log monitoring container for docker](https://blog.logentries.com/2015/07/an-all-inclusive-log-monitoring-container-for-docker/)

This tutorial will guide you through the **implementation of a very simple website**, the setup of a **web server running Docker** and the creation of a **single container** that **embed the full website**.

### Assumptions and definitions
For this tutorial, we will start with a fresh Ubuntu Server installation on Digital Ocean, but you can follow the same steps with any similar Ubuntu server. Please comment if you see any major difference.

![Digital Ocean logo](https://static-lab.engineor.com/Dockerizing-a-simple-website/digital-ocean.png)
![Ubuntu logo](https://static-lab.engineor.com/Dockerizing-a-simple-website/ubuntu.png)

We also consider you know HTML, CSS and Javascript enough to create a static website, or at least you can read the code we will provide.

Last requirement before we start, you must know the basic commands to connect to a Linux machine using SSH and browse folders / create files (including `ssh`, `cd`, `mkdir`, `touch` and `vim` or `nano`).

### Setting up the server

First thing first, let's create a `droplet` on Digital Ocean. This steps requires for you to have a Digital Ocean account, you can use this referal link to get one if you have to register: [get $10 free Digital Ocean credit](https://m.do.co/c/60774d3ab512).

We will use a standard `Ubuntu 14.04.4 x64` distribution (the server version), so you can create the same on your own machine or provider. Select the smallest instance size as we are only serving static content, and the datacenter region the closest to you. If you have an `ssh` key, please add it so no password are required to access the machine.

Once create, get on your instance details page, and copy the IP address. Use the following command to connect in your terminal or in putty for Windows users:

```ssh root@ip_address_you_just_copied```

If you added your `ssh` key, you should be connected. Else, please find out the password and access the server.

Now we will [install Docker](https://docs.docker.com/linux/step_one/): this step requires to add an extra repository to `apt` to get the latest version of Docker, then refresh and install. This is all combined in a script provided by Docker, so you only have to run the following command:

```
curl -fsSL https://get.docker.com/ | sh
```

**If you use a non-root user, which is recommanded, please read the script output, execute ``sudo usermod -aG docker `whoami` `` and logout / login again.**

Let's see if everything worked:

* `docker --version` gives something like `Docker version 1.10.3, build 20f81dd`
* `docker ps` gives `CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES`
* `docker run hello-world` gives the following output:

```
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
03f4658f8b78: Pull complete 
a3ed95caeb02: Pull complete 
Digest: sha256:8be990ef2aeb16dbcb9271ddfe2610fa6658d13f6dfb8bc72074cc1ca36966a7
Status: Downloaded newer image for hello-world:latest

Hello from Docker.
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker Hub account:
 https://hub.docker.com

For more examples and ideas, visit:
 https://docs.docker.com/userguide/
```

The server is now ready to run docker images containing our web application.

### Website implementation

As this is not the aim of the tutorial, we will keep the website as simple as possible.

Assuming you are in `/root` and logged in with the user `root` (use `pwd` and `whoami`), we will create the following files in **a new `src` folder** (`mkdir src`, then `vim src index.html` or `nano src/index.html`).

#### HTML: `index.html`

~~~html
<!Doctype html>
<html>
	<head>
		<meta charset="utf-8">
		<title>Dockerizing a simple website</title>
		<link rel="stylesheet" href="style.css">
	</head>
	<body>
		<h1>Dockerizing a simple website</h1>
		<script src="script.js"></script>
	</body>
</html>
~~~

#### CSS: `style.css`

~~~css
h1 {
	color: #34495e;
	text-align: center;
}
~~~

#### Javascript: `script.js`

~~~js
function ready(fn) {
  if (document.readyState != 'loading'){
    fn();
  } else {
    document.addEventListener('DOMContentLoaded', fn);
  }
}
ready(function() {
	alert('Dockerized?');
});
~~~

### Dockerizing

Docker offers `images`, immutable structures, in a way as would be the classes in an OOP code, and `containers`, mutable instances of specified classes.

`Images` are described using a `Dockerfile`, and can inherit from other images. Some free images are available on Docker Hub and can be pulled with not extra work required.

We will have to create an image first, embedding our static code, and then create a container from it. This container is supposed to be exposing its port 80 to the outside world (any client from outside the server).

#### Run a simple container
First step in this process will be running and exposing a simple container, instance of a public image supported by Docker. Because we want to answer to http connections, we will use a Nginx container.

Go to the [Docker Hub Nginx page](https://hub.docker.com/_/nginx/), and read the documentation.

Let's try to run it with the following command:

```
docker run --rm -p 80:80 nginx
```

* `run` is the action to execute.
* `--rm` indicates that the container has to be removed when stopped.
* `-p 80:80` forward the ports: the first one is the inside the container, the second is your main server.
* `nginx` is the name of the image available on Docker Hub.

The image will then be downloaded. Wait for it!

You should be stuck on `Status: Downloaded newer image for nginx:latest` if everything went well. The process is running in foreground, so you can't do anything else on the server for now.

Remember the ip address of your machine? Try to access it in your web browser. You should see the default Nginx page.

```
Welcome to nginx!

If you see this page, the nginx web server is successfully installed and working. Further configuration is required.

For online documentation and support please refer to nginx.org.
Commercial support is available at nginx.com.

Thank you for using nginx.
```

You can also have a look back to your ssh session: an error has been printed for a missing favicon file, and then your page has been served.

```
82.41.239.46 - - [02/Apr/2016:04:16:12 +0000] "GET / HTTP/1.1" 200 612 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/49.0.2623.110 Safari/537.36" "-"
2016/04/02 04:16:12 [error] 6#6: *1 open() "/usr/share/nginx/html/favicon.ico" failed (2: No such file or directory), client: 82.41.239.46, server: localhost, request: "GET /favicon.ico HTTP/1.1", host: "104.236.82.144", referrer: "http://104.236.82.144/"
82.41.239.46 - - [02/Apr/2016:04:16:12 +0000] "GET /favicon.ico HTTP/1.1" 404 571 "http://104.236.82.144/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/49.0.2623.110 Safari/537.36" "-"
```

Well done, we now know how to expose someone else's code... Press `ctrl+c` to stop the container, and because we used `--rf` it will completely disappear (try `docker ps`, it should not be there).

#### Add your own code
We have multiple solutions to add our own code here.

First, let's try to run the same container with an extra option to share some data from our host machine to the container. The `-v` option allows this precisely:

```
docker run --rm -p 80:80 -v `pwd`/src:/usr/share/nginx/html:ro nginx
```

*This command works assuming `pwd` is `/root`.*

`-v` allows to map directories from the host machine (first member) to the container (after the first column), and finally an optional column followed by the opening mode (read / write by default, `ro` is read only when specified).

Refresh your web browser page and you should see the website.

Use `ctrl+c` again to stop and remove the container and refresh your web browser to see that the page is not accessible anymore. 

For portability reasons, we do not want to share a code available on the server filesystem. What we actually want is embed and be able to distribute a Docker image that contains the whole code. Therefore, we will need to extend the Nginx image and add the code.

Create a file named `Dockerfile` (still assuming you are in `/root`), and add the following content:

```
FROM nginx:stable
MAINTAINER: Thomas Dutrion <thomas@engineor.com>

COPY src/ /usr/share/nginx/html
```

The first line indicates that we will extends the base image `nginx`, with the tag `stable` (tags after the columns allow you to specify the version of the image that you want. Then, the maintainer is just an information you give in order for others to contact you if they want to change anything or if they find you clever.

The last line however is more interesting. Two commands exist in a `Dockerfile` to add files to an image: `COPY` and `ADD`. In this file, we just want a plain copy from a local filesystem to the container filesystem, which is provided by `COPY` with no overhead. On the other hand, `ADD` is doing slightly more work, and allows to unarchive any added archive and add files from the web. Have a look at the documentation for further explained details: [Dockerfile best practices: ADD or COPY](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/#add-or-copy).

Now, we can build the image using our Dockerfile. Use the command `docker build -t test/my-static-website .` to do so.

`-t test/my-static-website` assign the name `test/my-static-website` to your image. Images on the local machine can be listed using `docker images`.
`.` at the end of the command says "find the `Dockerfile` in the current directory and build from it.

Here is the result of the `docker images` command:

```
REPOSITORY               TAG                 IMAGE ID            CREATED             SIZE
test/my-static-website   latest              3be8bb475bd8        15 seconds ago      133.3 MB
nginx                    stable              4b90b5603d01        3 weeks ago         133.3 MB
nginx                    latest              af4b3d7d5401        3 weeks ago         190.5 MB
hello-world              latest              690ed74de00f        5 months ago        960 B
```

As you can see, we have our new image on top of the list. Then we have `nginx` twice, because we did not specify any tag in the first commands we used, which resulted in using `latest`, but specified `stable` in the `Dockerfile`. We therefore need to choose only one and remove the unused ones. Use `docker ps -a` to see all the running and stopped containers. Nothing is running at the moment, we can therefore just remove Nginx stable: `docker rmi nginx:latest`. We can remove the Hello World container and image in the same time: `docker rmi -f hello-world `.

Using the command `docker history`, we can see the details on how the image is build for optimisation later on: `docker history test/my-static-website`.

```
IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
3be8bb475bd8        14 minutes ago      /bin/sh -c #(nop) COPY dir:756a5897ea96098ea4   541 B               
0bf59b18aafc        14 minutes ago      /bin/sh -c #(nop) MAINTAINER Thomas Dutrion <   0 B                 
4b90b5603d01        3 weeks ago         /bin/sh -c #(nop) CMD ["nginx" "-g" "daemon o   0 B                 
<missing>           3 weeks ago         /bin/sh -c #(nop) EXPOSE 443/tcp 80/tcp         0 B                 
<missing>           3 weeks ago         /bin/sh -c ln -sf /dev/stdout /var/log/nginx/   22 B                
<missing>           3 weeks ago         /bin/sh -c apt-key adv --keyserver hkp://pgp.   8.223 MB            
<missing>           3 weeks ago         /bin/sh -c #(nop) ENV NGINX_VERSION=1.8.1-1~j   0 B                 
<missing>           4 weeks ago         /bin/sh -c #(nop) MAINTAINER NGINX Docker Mai   0 B                 
<missing>           4 weeks ago         /bin/sh -c #(nop) CMD ["/bin/bash"]             0 B                 
<missing>           4 weeks ago         /bin/sh -c #(nop) ADD file:b5391cb13172fb513d   125.1 MB
```

You can see here the cost of each instruction, and optimise the image based on this data. This will be discussed in another tutorial later on.

Back to our current aim, we now have the image in our list, so we still have to create a container: `docker run -d --name my-static-website -p 80:80 test/my-static-website`. This will give you an output similar to `0445b0c32d57d859d8ef4a4507020369cb77f237e1ed9f537cb9519e2f25677b`.

As you can see, we used `-d` instead of `--rm` in the previous commands. This allows to run the container in background (deamon). `--name` allows us to define a name for the container that we will be able to use later to target this container. `docker ps` will give you an output as described thereafter:

```
CONTAINER ID        IMAGE                    COMMAND                  CREATED             STATUS              PORTS                         NAMES
80739c8b5851        test/my-static-website   "nginx -g 'daemon off"   6 seconds ago       Up 5 seconds        0.0.0.0:80->80/tcp, 443/tcp   my-static-website
```

You can access the website again in your web browser and see that it works as expected.

### Wrapping up

You have now seen different techniques for creating a Docker container containing your static website and how to run it on your server. The image we created can be reused on any Docker host, which will be the object of another tutorial.

The next step for us will be to deploy a PHP application that has the ability to send emails, as it is a current use case for any showcasing website.