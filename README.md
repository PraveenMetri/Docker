# Docker-Commands
Docker-ppt


Welcome to the Docker-Commands wiki!

Docker.

What docker can do for you.
Dev team has same environment 
DEV, QA and PROD environment are in sync 
Desktop and cloud easy deployment and scaling 
Build, fast and better 

Why to use Docker

Installing Docker 

# Yum install docker.io -------> Installing docker on a host

#Docker -version -------> List docker version and client version 

# docker ------->  list the all possible commands for docker 

# docker -it run --name test1 centos:latest /bin/bash  -------> create a docker container in 
interactive mode with centos image, first it will check the image in local repo OR it goes to docker hub and pulls it.

# docker images -------> List of docker images which are in local machine.

# docker ps -a -------> List of docker container running or exited.

# docker rm test-container -------> Deletes the container 


Creating docker image -- dockerfile for running sshd daemon inside Docker Container.

Creating a Dockerfile

# Vi Dockerfile -------> Name of the docker file should be same 

FROM centos:latest -------> creating a image from centos 
MAINTAINER Praveen@gmail.com -------> Maintainer of the image
RUN yum update
RUN yum install openssh-server -y  -------> Installing openssh 
RUN cp  /etc/ssh/sshd_config /etc/ssh/sshd_config.original -------> Copying the original file for safer side
RUN chmod a-w /etc/ssh/sshd_config-original -------> Changing the permission
RUN mkdir /var/run/sshd 
RUN echo ‘root:yourpassword’ | chpasswd -------> Changing the root passsword
RUN sed -i ‘s/PermitRootLogin without-password/PermitRootLogin yes/’ /etc/ssh/sshd_config ------->permitting root to login and changing the config file entry
CMD [“/usr/sbin/sshd”, “-D””] -------> This is the default command run whenever this image runs
EXPOSE 22 -------> Exposing the 22 port for container

 # docker build -t localreopo/centossshd:v1 . -------> Building a image from dockerfile

# docker images -------> List all images 


Testing 

# docker run -d -P --name testsshd repo/centos:v1

# docker ps

Connecting to testsshd container through SSH
# ssh root@localhost -p 49153

Asks for root password then you can login to the testsshd container 

# docker stop testsshd

# docker ps -a 

# docker start testsshd

# docker rm testsshd


Creating Docker file for apache2

# Vi Dockerfile -------> Name of the docker file should be same 

FROM ubuntu:14.04 -------> creating a image from ubuntu
MAINTAINER Praveen@gmail.com -------> Maintainer of the image
RUN apt-get update
RUN apt-get install apache2 -y  -------> Installing openssh 
RUN mkdir /var/lock/apache2
ENV APACHE_RUN_USER www-data
ENV APACHE_RUN_GROUP www-data
ENV APACHE_PID_FILE  /var/run/apache2.pid
ENV APACHE_RUN_DIR /var/run/apache2
ENV APACHE_LOCK_DIR /var/lock/apache2
ENV APACHE_LOG_DIR /var/log/apache2
CMD [“/usr/sbin/apache2”, “-D”, “FOREGROUND”]
EXPOSE 80


# docker build -t localreopo/ubuntuapahce2:v1 .

# docker images 

# docker run -d -P --name myweb localrepo/ubuntuapache2:v1 -------> while running this docker container port 80 of the container will be mapped to on host with some random port, so we can access the host port in browser to verify the container has created properly.

# docker ps 

# http://localhost:41936 -------> Should show apache2 default page.


Mounting source code inside Docker Container from host machine.

# docker run -d -P -v /home/ec2-user/mycode:/var/www/html --name mywebpage localrepo/ubuntuapache2:v1 -------> -v is for mounting local directory from host machine to container 

# docker ps -a 

# http://localhost:41937  -------> Should show the content of index.html

# docker inspect mywebpage -------> We can check the info of container like IP address and mounting point 


How to use supervisor to run/manage multiple processes inside a Docker container

There should be only one CMD can run inside a container
Only one process can be running in a container in FOREGROUND
If you want two services should be running in a container for example opensshd and apache, then you can use SuperVisor
SuperVisor is processor management tool for container

How to use supervisor with Docker for running multiple processer in a container

# vi supervisord.conf

[supervidord]
nodaemon=true

[program:sshd]
command=/usr/sbin/sshs -D

[program:apache2]
command=/bin/sbin -c “source /etc/apache2/envvars && /usr/sbin/apache2 -DFOREGROUND”


# vi Dockerfile

FROM centos:latest -------> creating a image from centos 
MAINTAINER Praveen@gmail.com -------> Maintainer of the image
RUN yum update && yum install openssh-server -y  apache2 supervisor-------> Installing openssh, apache and supervisor
RUN mkdir /var/run/sshd /var/lock/apache2 /var/run/apache2 /var/log/supervisor
RUN cp  /etc/ssh/sshd_config /etc/ssh/sshd_config.original -------> Copying the original file for safer side
RUN chmod a-w /etc/ssh/sshd_config-original -------> Changing the permission
RUN echo ‘root:yourpassword’ | chpasswd -------> Changing the root passsword
RUN sed -i ‘s/PermitRootLogin without-password/PermitRootLogin yes/’ /etc/ssh/sshd_config ------->permitting root to login and changing the config file entry
COPY supervisord.conf /etc/supervisor/conf.d/supervisor.conf
EXPOSE 22 80 -------> Exposing the 22 and port 80 for container
CMD [“/usr/bin/supervisord”] -------> This is the default command run whenever this image runs

# docker build -t localrepo/supervisorapp-image .

# docker images

# docker run -d -P -v /home/ec2-user/mycode:/var/www/html --name supervisor-webapp supervisorapp-image

 # docker ps -l 

Testing

# ssh root@localhost -p 46931
Check web servers is running fine by checking from webbrowser

http://localhost:46932


How to link Docker containers together (mysql + webserver)

Creating mysql container 

#docker run --name mysqldb -e MYSQL_ROOT_PASSWORD=password -d mysql:5.5

# docker ps -l

Creating container with web application

# docker run --name mylinkedweb --link mysqldb:dbalias -d localrepo/ubunutuapache2.v1 env -------> here “env” will list all the environment variables and linking the running mysqldb container to ubuntuapache2 container 

When ever there the source container is restarted then IP of that container will be changes, so if that container is linked to any other running container, docker container will map the new Ips to the respective linked container.


How to setup your wordpress using Docker

Creating a mysql container for wordpress

# docker run --name mysqldb -e MYSQL_ROOT_PASSWORD=password -d mysql:5.5

Running wordpress container and linking to running mydqldb container 

# docker run -- name mywordpress --link mysqldb:mysql -P -d wordpress

Testing 

# docker ps -l 

Check all wordpress ports are mapped to host machine, then check in web browser to confirm

http://localhost:49613  -------> should display wordpress configuration page.


Docker Best Practices
Comment in Dockerfile
Don't install all unnecessary packages in one container.
Each RUN instruction create a layer on a image and there is layer limit for a image, so make sure that you have less number of layers.
While creating a image from Dockerfile make sure that the path from where Dockerfile is there, it will update all the files from that path to the image.
