# Maven
### An example of deployment to nexus via maven-deploy-plugin
#### Method 1
```console
foo@bar:~$ mvn deploy:deploy-file \
               -Durl=http://172.20.17.14:8081/repository/maven-snapshots/ \
               -Dfile=target/helloworld.war \
               -DgroupId=org.jboss.as.quickstarts.helloworld \
               -DartifactId=helloworld \
               -Dpackaging=war \
               -Dversion=1.15-SNAPSHOT \
               -DrepositoryId=maven-snapshots
```
NEEDED: $HOME/.m2/settings.xml:
```xml
<settings>
    <servers>
        <server>
            <id>maven-snapshots</id>
            <username>admin</username>
            <password>test</password>
        </server>
    </servers>
</settings> 
```
#### Method 2
maven-snapshots and maven-releases are corresponding (to settings.xml) repository ids here
```console
foo@bar:~$ mvn -X \
               -DaltSnapshotDeploymentRepository=maven-snapshots::default::http://172.20.17.14:8081/repository/maven-snapshots/ \
               -DaltReleaseDeploymentRepository=maven-releases::default::http://172.20.17.14:8081/repository/releases/ \
               deploy'
```
NEEDED: $HOME/.m2/settings.xml (THE EXAMPLE SAME AS METHOD 1)
#### Method 3
```console
foo@bar:~$ mvn -Drepo.login=admin -Drepo.pwd=test deploy
```
$HOME/.m2/settings.xml
```xml
<settings>
	<profiles>
		<profile>
			<id>nexus</id>
			<properties>
				<altSnapshotDeploymentRepository>maven-snapshots::default::http://172.20.17.14:8081/repository/maven-snapshots/</altSnapshotDeploymentRepository>
				<altReleaseDeploymentRepository>maven-releases::default::http://172.20.17.14:8081/repository/releases/</altReleaseDeploymentRepository>
			</properties>
		</profile>
	</profiles>
	<activeProfiles>
		<activeProfile>nexus</activeProfile>
	</activeProfiles>
	<servers>
		<server>
			<id>maven-releases</id>
			<username>${repo.login}</username>
			<password>${repo.pwd}</password>
		</server>
		<server>
			<id>maven-snapshots</id>
			<username>${repo.login}</username>
			<password>${repo.pwd}</password>
		</server>
	</servers>
</settings>  
```
#### Method 4
```console
foo@bar:~$ mvn -Drepo.login=admin \
               -Drepo.pwd=test \
               -Drepo.release.url=http://172.20.17.14:8081/repository/releases/ \
               -Drepo.release.id=maven-releases \
               deploy
```
$HOME/.m2/settings.xml
```xml
<settings>
	<servers>
		<server>
			<id>maven-releases</id>
			<username>${repo.login}</username>
			<password>${repo.pwd}</password>
		</server>
		<server>
			<id>maven-snapshots</id>
			<username>${repo.login}</username>
			<password>${repo.pwd}</password>
		</server>
	</servers>
</settings>  
```
In pom.xml you can use variables or plain text. In properties field you can set the default parameters. <br>
$PROJECT/$PACKAGE/pom.xml  <-- YOU BUILD FROM HERE
```xml
    <properties>
        <repo.release.id>maven-releases</repo.release.id>
        <repo.release.url>http://172.20.17.14:8081/repository/releases/</repo.release.url>
        <repo.snapshots.id>maven-snapshots</repo.snapshots.id>
        <repo.snapshots.url>http://172.20.17.14:8081/repository/maven-snapshots/</repo.snapshots.url> 
    </properties>
    <distributionManagement>
        <repository>
            <id>${repo.release.id}</id>
            <url>${repo.release.url}</url>
        </repository> 
        <snapshotRepository>
            <id>maven-snapshots</id>
            <url>http://172.20.17.14:8081/repository/maven-snapshots/</url>
        </snapshotRepository> 
    </distributionManagement>
```
