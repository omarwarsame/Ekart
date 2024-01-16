**Project\_11**  (Repo: <https://github.com/jaiswaladi246/Ekart.git> )

**Create 3 ubuntu EC2’s with 20gb storage (call them Jenkins, SonarQube and Nexus)**

**1. For Jenkins:**

1. sudo su -

2. sudo apt update

3. sudo apt install openjdk-17-jre-headless -y

4. apt  install docker.io -y

5. chmod 666 /var/run/docker.sock

6. sudo hostnamectl set-hostname jenkins

7. /bin/bash

Install Jenkins

`sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \`

      https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
    echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
      https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
      /etc/apt/sources.list.d/jenkins.list > /dev/null
    sudo apt-get update
    sudo apt-get install jenkins

Do the next 3 steps separately:

sudo systemctl enable jenkins

sudo systemctl start jenkins

sudo systemctl status jenkins

Login to jenkins: Server IP + 8080 (port  must be open in AWS)

password is in  /var/lib/jenkins/secrets/initialAdminPassword

Install the Suggested plugins

**2. For SonarQube: (Remember SonarQube will be a container)**

1. sudo su -

2. sudo apt update

3. sudo hostnamectl set-hostname sonarqube

4. /bin/bash

5. apt  install docker.io -y

6. docker run -d -p 9000:9000 sonarqube:lts-community

7. login to sonarqube serve (on server IP  + 9000) with  admin admin and change password

**3. For Nexus: (Remember Nexus will be a container)**

1. sudo su -

2. sudo apt update

3. sudo hostnamectl set-hostname nexus

4. /bin/bash

5. apt  install docker.io -y

6. docker run -d -p 8081:8081 sonatype/nexus3

Do docker exec -it into  the container and  find the password in **/nexus-data/admin.password**

Login to nexus with admin and the password you’ve just found. Then Enable anonymous access

All 3 Servers installations are done!

Next, we will configure them even more!

**Configure plugins in Jenkins:**

Manage Jenkins > Available plugins and install these:

1. Sonarqube Scanner (This will do the code analysis)

2. Nexus Artifact Uploader

3. Docker

4. Docker Pipeline

5. CloudBees Docker Build and Publish

6. Docker-build-steps

7. OWASP Dependency-Check

8. Eclipse Temurin Installer (For installing multiple JDK versions)

9. Config File Provider 

Configure the tools in jenkins

In _Manage Jenkins > Tools >_ 

Find JDK installation:

Name= jdk17

Check _Install automatically_

In the dropdown list, find _Install from adoptium.net_

Select version _jdk-17.0.9+9_

Add another  JDK, call it jdk11 and select version _jdk-11.0.21+9_

Scroll down and find SonarQube section

Click on _Add SonarQube Scanner >_ Name it sonar-scanner and leave it with the latest version

Scroll down and find Maven > Call it maven3, select version 3.6.3

Scroll down and find Dependency-Check installations > call it _dependency-check_  

Check install automatically > Install from github.com > Version _dependency-check 6.5.1_ 

Scroll down to Docker, Name it _docker_ , Install automatically and select Download from docker.com > latest. 

Apply and save.

**Configure SonarQube**

Generate a token > Administrator >  Security > Users > Click on the burger icon on right to update tokens

Call it ‘token’ Make it expire in 30 days, Click ‘generate’. Copy the token from screen and save it. Click ‘done’

\_\_

Go back to jenkins:

In _Manage Jenkins > Credentials > global > Add credentials > Secret Text > Paste token from SonarQube, give it ID._

Add another Credentials for docker user

Type: Username with password

Put username and password and give an ID as docker-cred

\_\_\_\_\_\_\_

Configure Servers

Go to Manage Jenkins  > System > 

Scroll down to find sonarqube server (This is only available after you’ve installed its plugin)

Click on Add SonarQube > Call  it sonar-server >  put the url of the server (with port 9000)  in  the next box.  

Scroll down to the token and put the token you just created in the credential section.  Apply and save. 

Go to Go to Manage Jenkins > Manage files > Add a new Config > Global Maven settings.xml > 

Give an ID of ‘global-maven’ Click on next.  In  the next window, Find and uncomment the next script :

\<server>

      \<id>maven-releases\</id>

      \<username>admin\</username>

      \<password>123\</password>

\</server>

_Copy those 5 lines under the one you just edited and make it like this:_

\<server>

      \<id>maven-snapshots\</id>

      \<username>admin\</username>

      \<password>123\</password>

\</server>

These were entries from your nexus (server administration and configuration > repositories then 

We will come back to this and fill in with some details.

Go to your Nexus server, find maven-releases and maven-snapshots copy both ID’s (

\_\_\_

Configure Nexus Server
