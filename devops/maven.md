# Maven
### An example of deployment to nexus via maven-deploy-plugin
````
mvn deploy:deploy-file -Durl=http://172.20.17.14:8081/repository/maven-snapshots/ -Dfile=target/helloworld.war -DgroupId=org.jboss.as.quickstarts.helloworld -DartifactId=helloworld -Dpackaging=war -Dversion=1.15-SNAPSHOT -DrepositoryId=maven-snapshots
````
NEEDED:
$HOME/.m2/settings.xml:
````
<settings>
    <servers>
        <server>
            <id>maven-snapshots</id>
            <username>admin</username>
            <password>test</password>
        </server>
    </servers>
</settings> 
````
NOT NEEDED:
$(pwd)/pom.xml:
````
...
    </pluginRepositories>
    <distributionManagement>
        <repository>
            <id>maven-snapshots</id>
            <url>http://172.20.17.14:8081/repository/maven-snapshots/</url>
        </repository>
    </distributionManagement>
    <build>
...
````
