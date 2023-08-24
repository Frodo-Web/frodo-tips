# Jenkins
## CLI
````
// Check auth
$ java -jar jenkins-cli.jar -auth frodo:test -s http://localhost:8080 who-am-i
..
Authenticated as: frodo
Authorities:
  authenticated
// OR
$ export JENKINS_USER_ID=frodo
$ export JENKINS_API_TOKEN=...
$ java -jar jenkins-cli.jar -s http://localhost:8080 who-am-i
..
Authenticated as: frodo
Authorities:
  authenticated
 
$ java -jar jenkins-cli.jar -s http://172.20.17.11:8080/ help                       // Show command-line options
$ java -jar jenkins-cli.jar -auth frodo:test -s http://172.20.17.11:8080/ help      // With auth, lists all available commands of the cli http://172.20.17.11:8080/manage/cli/

// Get job, create job
$ java -jar jenkins-cli.jar -auth frodo:test  -s http://localhost:8080 get-job JenkinsJob-1 > JobTemplate.xml
..
  <concurrentBuild>true</concurrentBuild>
  <builders>
    <hudson.tasks.Shell>
      <command>echo &quot;Hello world&quot;
echo &quot;Build number is $BUILD_NUMBER&quot;
pwd
whoami
echo &quot;Pause for 5 sec&quot;
sleep 5s
echo &quot;BUILD_DISPLAY_NAME : $BUILD_DISPLAY_NAME&quot;</command>
      <configuredLocalRules/>
    </hudson.tasks.Shell>
  </builders>
...
// ... We make changes to that file
$ java -jar jenkins-cli.jar -auth frodo:test  -s http://localhost:8080 create-job NewJob < JobTemplate.xml
````
## Script Console
````
println System.getenv("PATH")
println "uname -a".execute().text

// Write a file
new File('/tmp/file.txt').withWriter('UTF-8') { writer ->
    try {
        writer << 'hello world\n'
    } finally {
        writer.close()
    }
}

// Read file
new File('/tmp/file.txt').text

// Write file from master to agent throught agent channel
import hudson.FilePath
import hudson.remoting.Channel
import jenkins.model.Jenkins

String agentName = 'jenkins_agent'
String filePath = '/tmp/file.txt'

Channel agentChannel = Jenkins.instance.slaves.find { agent ->
    agent.name == agentName
}.channel

new FilePath(agentChannel, filePath).write().with { os ->
    try {
        os << 'hello world\n'
    } finally {
        os.close()
    }
}

// Execute groovy scripts remotely (which are located on the host)
curl -d "script=<your_script_here>" https://jenkins/script
curl -d "script=<your_script_here>" https://jenkins/scriptText  // Plain text instead of HTML
// Execute local groovy scripts 
curl --user 'username:api-token' --data-urlencode \
  "script=$(< ./somescript.groovy)" https://jenkins/scriptText
````
## Pipeline
Multi-line shell commands:
````
pipeline {
    agent any

    stages {
        stage('Stage One') {
            steps {
                sh '''
                    echo "Line 1"
                    echo "Line 2"
                '''
            }
        }
    }
}
````
Run stages inside container:
````
pipeline {
    agent { docker { image 'python:latest' }}

    stages {
        stage('Stage One') {
            steps {
                sh '''
                    echo "Line 1"
                    echo "Line 2"
                '''
            }
        }
    }
}
````
## Setup docker in docker to run stages inside containers
### First
You need to make /var/run/docker.sock the general file for both host and container
````
volumes:
      - /var/run/docker.sock:/var/run/docker.sock
````
The whole docker-compose.yml:
````
version: '3.9'

services:

  jenkins:
    image: jenkins/jenkins:lts-jdk11
    container_name: jenkins
    hostname: jenkins
    ports:
      - 8080:8080
      - 50000:50000
    volumes:
      - /home/dev/Practice/Jenkins/jenkins-data:/var/jenkins_home
    environment:
      TZ: "Europe/Moscow"
    networks:
      jenkins:
        ipv4_address: 172.20.17.11

  jenkins-agent:
    image: jenkins/ssh-agent:jdk11
    container_name: jenkins-agent
    hostname: jenkins-agent
    ports:
      - 22:22
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      TZ: "Europe/Moscow"
      JENKINS_AGENT_SSH_PUBKEY: "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDOyVfvYGdsnkNR6X7XxwL5/CCE3i183+Ah096VVTEkIiukd6zv3EtWWoSK+7rpnzZZCta92egxxWoZ8NZFAy6gusUcyOL1iLAtoLojHqWbrmonsHYRECu7YNiRcSKwJCNg9ukrBNraipZNogE3/bnMQxeZ5gbZJ2KrbTJls0sRDrGK63uPZFRQZkz9i8jDwJhIGjl6pAW7rmOgYtVYWGgmBQt016uEOfBNI4CX8fGn5ISmlEfi6bZ5PZIWwKcnuwsrxJQ0wh6m/pvg4HHjaqTGubdus5vG6pTJt6d1qoFnEyKXwbhUzNUY2iu6d/tM/XC1CekVb3BXhj9W7cpxFf3G48CWZAJ90yWSGAXi56RN9eZhVFBAL+2gqXidoAd1paeNg3Lagq7C/nh78Fqmb+tKXV+3GiTnukZRkRaAQwsaxtX7mrp/XiwWdYZSsv8lqph8H5b1CziQTlnH+uFVErgtO9VFbFu8WZTIZblkk2Jdktb/1YMpX+ljMN6tQJW0BxM= dev@Lustre"
    networks:
      jenkins:
        ipv4_address: 172.20.17.12

networks:
  jenkins:
    name: network_jenkins
    driver: bridge
    ipam:
      driver: default
      config:
        - subnet: 172.20.17.0/24
          ip_range: 172.20.17.0/24
          gateway: 172.20.17.1
    driver_opts:
      com.docker.network.bridge.name: br_jenkins
````
### Second
Install docker on that agent in which the pipeline will run (jenkins-agent in my case):
````
$ docker exec -u 0 -it jenkins-agent /bin/bash
# apt install docker docker.io docker-compose containerd
````
### Third
Add docker group to jenkins user on the agent. 
````
usermod -a -G docker jenkins
````
And make sure docker group id on the agent matches host's docker group id.
````
$ less /etc/group
$ docker exec -u 0 -it jenkins-agent /bin/bash
# vim /etc/group
...
docker:x:997:jenkins
````
### Fourth
Restart docker compose, try to run the pipeline
````
$ Ctrl + C
$ docker-compose up
````
