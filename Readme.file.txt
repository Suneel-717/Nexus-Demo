									ARTIFACT REPOSITORY TOOL - NEXUS
													by
												Kastro Kiran V
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------

------------
Demo
-------------
In this demo i will show how to setup Nexus and Jenkins, and then we will see how to push the Artifacts into the Nexus Repo.

Step 1: Launch Ubuntu VM (24.04), t2.large, EBS: 28GB
Open the below ports (optional)
25, 30000-100000, 80, 443, 587, 22, 6443, 30000-32767, 465, 27017

Step 2: Connect to the VM using MobaXTerm

Step 3: Lets setup the Jenkins
$ sudo apt update
$ sudo apt install openjdk-17-jre-headless -y
$ vi jenkins.sh ---> Paste the below commands in the file ---> 

sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins -y

 ---> esc  ---> :wq

Lets give the permission to jenkins.sh file to execute; sudo chmod +x jenkins.sh
ls  ---> You can see 'jenkins.sh' in green colour which means the file is executable

Lets execute the jenkins.sh file; ./jenkins.sh
with this Jenkins installation is done.
Note that, Jenkins will run on 8080 port number by default.

Open Port No. 8080 for the VM

Access Jenkins; <publicip>:8080 ---> Setup the Jenkins dashboard by following the known steps

Step 4: Lets setup the Docker. And then using Docker, we will setup Nexus.
URL: https://docs.docker.com/engine/install/ubuntu/

vi docker.sh ---> Paste the below content ---> 

# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

 ---> esc ---> :wq ---> 
Lets give the permission to docker.sh file to execute; sudo chmod +x docker.sh
ls  ---> You can see 'docker.sh' in green colour which means the file is executable

Lets execute the docker.sh file; ./docker.sh

Install the Docker packages - To install the latest version;
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

sudo su

docker --version ---> You can see the docker version

Step 5: Lets setup Nexus using Docker image

docker run -d --name nexus3 -p 8081:8081 sonatype/nexus3
#in the above command, 
#nexus3 is the name of the container. any name can be given
#8081 is the default port number of nexus
#sonatype/nexus3 is the docker image of nexus. Reference URL: https://hub.docker.com/r/sonatype/nexus3/

docker ps ---> you can see the container running

Open Port number 8081 for VM

Access Nexus; <publicip>:8081 ---> Nexus will take sometime to run. Initially you might see, 'This site cant be reached' wait for sometime
After sometime reload the page, you will see the nexus home page.

To login to nexus, click on 'Signin' in the top right corner. You can see the username as admin. But to get the password, a path is mentioned.
To get the password; goto the MobaXTerm; now we need to go inside the container we have created sometime back;

To go inside a container;
$ docker exec -it <ContainerID> /bin/bash ---> You are now inside the container ---> ls ---> You can see the nexus related files ---> cd sonatype-work/ ---> cd nexus3/ ---> You can see 'admin.password' file ---> cat admin.password ---> Copy the password, dont copy 'bash-4.4' ---> Goto the nexus connected tab in browser ---> 
Username: admin
Password: Paste the password copied from MobaXTerm
 ---> set new password ---> 'check' enable anonymous access ---> Next ---> Finish

Goto MobaXTerm ---> Come out of the terminal ---> exit

Note: Please follow the next steps as discussed in the video.

Git Repo URL for Nexus Code: https://github.com/KastroVKiran/Nexus-Demo.git
-----------------
Final Script
-----------------
pipeline {
    agent any
    tools {
        maven 'maven3'
    }
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/KastroVKiran/Nexus-Demo.git'
            }
        }
        stage('Compile') {
            steps {
                sh 'mvn compile'
            }
        }
        stage('Tests') {
            steps {
                sh 'mvn test'
            }
        }
        stage('Build') {
            steps {
                sh 'mvn package'
            }
        }
        stage('Deploy Artifacts') {
            steps {
                withMaven(globalMavenSettingsConfig: 'settings.xml', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh 'mvn deploy'
                }
            }
        }
    }
}

---> Apply ---> Save ---> Build the job
