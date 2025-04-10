# Project 4 CI 

## Part 1 DOCKERIZE

### Docker setup

<p>Download docker desktop. Since I'm using WSL2, when installing docker desktop, configure WSL2 dependency and make sure the tab in the GUI is flipped on for ubuntu. Then run the followin commands in order:</p>

```
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  
  # following command is nested inside previous command
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

<p>verify installation was successful with following command</p>

`sudo docker run hello-world`

### Setting up my containers && Dockerfile && Building images

<p>The CRUD app I've designed and am using for this project requires a mySQL server, so the first step is creating a container for mySQL. I did this by first downloading the docker image for
mySQL from docker hub using</p>

`docker pull mysql:latest`

<p>I then run the following command to construct a container for my database NOTE: your command will be different, this includes the name of my database</p>

`docker run -d -p 3308:3306 --name=mysql-docker --env="MYSQL_ROOT_PASSWORD=root" --env="MYSQL_PASSWORD=root" --env="MYSQL_DATABASE=mtg_collection_tracker_db" mysql`

<p>This command builds the container, binds it outwardly to port 3308 and inwardly to port 3306(mysql default port), names the container "mysql-docker", username root password root and the name of the database connecting to based off the application.properties file in my spring boot app (picture to follow)</p>

![application.properties](./appprops.PNG)

<p>Then with container built I use the following commands to go inside and make sure the tables are correct</p>

```
docker exec -it mysql-docker bash
    mysql -uroot -proot
        SHOW TABLES;
```

<p>Tables display and verify that mySQL container is built properly. Next is building the container for my Java app. First step was exporting Java app to a .jar file for the Dockerfile to use.
This is done by using the following command in windows powershell</p>

`mvn clean package`

<p>This generates your .jar file which is then used in your Dockerfile. My dockerfile looks like this:</p>

![Dockerfile](./dockerfile.PNG)

<p>I then use the following commands to first build the container, then run it on port 5000 and connect it to the mysql container</p>

```
docker build -t mtgcollectionapp-image .
docker tag <your container ID from docker desktop or docker image -a here> timmonsevan/timmons-ceg3120
docker run -t --link mysql-docker:mysql -p 5000:5000 mtgcollectionapp-image
```

<p>from the container side I am seeing green lights on docker desktop, no build errors on the command line and on the host side when I go to localhost:5000 in my browser, I see the app running!</p>

### DockerHub Repo

<p>To setup a PAT, I go to > account settings > personal access tokens, click generate new token and follow the two step procedure thereafter to generate it and then connect with it using</p>

`docker login -u <my username>`

<p>and then pasting the token into the password field that follows on the command line. Then to push my container image to my DockerHub repo, I used the following command:</p>

`docker push timmonsevan/timmons-ceg3120:latest`

[dockerhub repo](https://hub.docker.com/repository/docker/timmonsevan/timmons-ceg3120/general)

<p>The credentials to login to my app after running are username: root password: toor</p>
