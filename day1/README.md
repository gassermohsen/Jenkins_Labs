### Jenkins Day 1 

1- install jenkins with docker image 
```bash
docker run -p 8081:8080 -d jenkins/jenkins:lts
```
2- Install role based authorization plugin 
![Screenshot](screenshots/1.png)

3- Create new user

![Screenshot](screenshots/2.png)

4- create read role and assign it to the new user

![Screenshot](screenshots/3.png)
____

![Screenshot](screenshots/4.png)

5-  create free style pipeline and link it to private git repo(inside it create directory and create file with "hello world") 

![Screenshot](screenshots/5.png)

___
![Screenshot](screenshots/6.png)

6- create declarative in jenkins GUI pipeline for your own repo to do "ls"

- First add the git credentials 
![Screenshot](screenshots/7.png)

- Second add the jenkins file on github

![Screenshot](screenshots/8.png)

- Build results

![Screenshot](screenshots/9.png)

7-Create scripted in jenkins GUI pipeline for your own repo to do "ls"

- Scripted Code 
![Screenshot](screenshots/10.png)

8- Create the same with jenkinsfile in your branches as multibranch pipeline

- Scan repository Log
![Screenshot](screenshots/11.png)

- All Branches With jenkins file

![Screenshot](screenshots/12.png)









