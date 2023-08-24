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
