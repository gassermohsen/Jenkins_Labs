### Jenkins Day2 

##### 1- Configure jenkins image to run docker commands on your host docker daemon

- First Build jenkins Docker image with docker client installed on it 

```dockerfile 
FROM jenkins/jenkins:lts
USER root
RUN apt-get update -qq \
    && apt-get install -qqy apt-transport-https ca-certificates curl gnupg2 software-properties-common
RUN curl -fsSL https://download.docker.com/linux/debian/gpg | apt-key add -
RUN add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/debian \
   $(lsb_release -cs) \
   stable"
RUN apt-get update  -qq \
    && apt-get -y install docker-ce
RUN usermod -aG docker jenkins
```
- Second run docker container with the docker image we created and mount volume of the daemon from your host 

```bash
docker run -it -p 8081:8080 -v /var/run/docker.sock:/var/run/docker.sock -v jenkins_home:/var/jenkins_home jenkins-docker
```



##### 2- Create CI/CD for this repo https://github.com/mahmoud254/jenkins_nodejs_example.git

- Create pipeline in jenkins with the following code 

```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                // Get some code from a GitHub repository
                git 'https://github.com/mahmoud254/jenkins_nodejs_example.git'

                // Run Maven on a Unix agent.
                sh "docker build -f dockerfile . -t jenkins-node-app"
                sh "docker run -d -p 3000:3000 jenkins-node-app"

                // To run Maven on a Windows agent, use
                // bat "mvn -Dmaven.test.failure.ignore=true clean package"
            }

            post {
                // If Maven was able to run the tests, even if some of the test
                // failed, record the test results and archive the jar file.
                success {
                    sh "echo Success"
                }
            }
        }
    }
}
```


##### 3- Create docker file to build image for jenkins slave 


- Create Dockerfile
```dockerfile
#From ubuntu base image install the following 
FROM ubuntu 
USER root
RUN apt-get update -qq
RUN mkdir -p jenkins_home
RUN apt install openjdk-11-jdk -qq
RUN apt install openssh-server -qq
RUN service ssh start -qq

# Install Docker client 

RUN apt-get update -qq \
    && apt-get install -qqy apt-transport-https ca-certificates curl gnupg2 software-properties-common
RUN curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
RUN add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
RUN apt-get update  -qq \
    && apt-get -y install docker-ce

CMD ["/bin/bash"]

```

##### 4- Create container from this image and configure ssh 

- Build this image 

```bash
docker build . -t Docker-slave
```

- After that you need to create keygen but not from the docker container because it will not work , create it from your host and copy the pub key to your authorized_keys file on the Slave Container 

- Run the container and mount the daemon volume 

```bash
docker run -it -p 8081:8080 -v /var/run/docker.sock:/var/run/docker.sock -v jenkins_home:/var/jenkins_home Docker-slave
```

##### 5-From jenkins master create new node with the slave container

- Now you need to configure new node in jenkins and add all the slave information.

----
- Step One : Manage Jenkins > Manage Nodes and clouds > New Node 

- Step Two : Fill all the info you need and add the Remote root directory to the directory we created in the Dockerfile 
  
- Step Three : Create Credentials with the ssh connection and add the private key you created from the host 

> **_NOTE:_**  You have to inspect the container to get the ip and in the username write root ! 
 


##### 6- Integrate slack with jenkins 

- Install slack plugin 
- Configure jenkins on slack workspace

![Screenshot](screenshots/1.png)


##### 7- Send slack message when stage in your pipeline is successful

![Screenshot](screenshots/2.png)


##### 8- Install audit logs plugin and test it 

![Screenshot](screenshots/3.png)
![Screenshot](screenshots/4.png)

##### 9- Fork the following repo https://github.com/mahmoud254/Booster_CI_CD_Project and add dockerfile to run this django app and use github actions to build the docker image and push it to your dockerhub

- Add docker hub account id and password as a secret in github 
- Add docker file with the following code 

```Dockerfile
# Use an official Python image as the base image
FROM python:3.9

# Set the working directory in the container to /app
WORKDIR /app

# Copy the requirements.txt file to the container
COPY requirements.txt .

# Install the required packages
RUN pip install -r requirements.txt

# Copy the rest of the application files to the container
COPY . .

# Set environment variables for the Django app
ENV DJANGO_SETTINGS_MODULE=myproject.settings
ENV PYTHONUNBUFFERED=1

# Run migrations and collectstatic files
RUN python manage.py 

# Start the Django development server on port 8000
CMD ["python", "manage.py"]
``` 

- Go to github actions and add this action code 

```yaml

name: Docker Image CI

on:
  push:
    branches: [ "master" ]
  pull_request:
    branches: [ "master" ]

jobs:

  build:

    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v3
    - name: docker login
      env:
        DOCKER_USER: ${{secrets.DOCKER_USER}}
        DOCKER_PASSWORD: ${{secrets.DOCKER_PASSWORD}}
      run:
        docker login -u $DOCKER_USER -p $DOCKER_PASSWORD
    - name: Build the Docker image
      run: docker build . -f Dockerfile -t ${{secrets.DOCKER_USER}}/jenkins_lab2
    - name: Docker push
      run: docker push ${{secrets.DOCKER_USER}}/jenkins_lab2
```

--- 

![Screenshot](screenshots/5.png)

---
- Image Pushed to my docker hub 
![Screenshot](screenshots/6.png)

