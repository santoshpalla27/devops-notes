# Nexus Artifactory Notes

Artifactory is used to store the artifacts after they are build there are different types of artifactory they are nexus,jfrog etc

```xml
<version>2.246-SNAPSHOT</version>  <!-- this will decide where the war file go to the snapshot or release -->
<version>2.246</version>  <!-- pom.xml -->
```

Install maven in system and not include in tools if username and password configured manually in vm and include maven tool if configured through Jenkins config management for username and pass

## Creation of Nexus Server

Make a fresh ec2 instance with t2.medium t2.micro doesn't work

The default port number for Nexus Artifactory is 8081

First install java specially

```bash
wget https://download.java.net/java/GA/jdk11/openjdk-11_linux-x64_bin.tar.gz

tar -xzf openjdk-11_linux-x64_bin.tar.gz

sudo mv jdk-11 /usr/local/

export JAVA_HOME=/usr/local/jdk-11
export PATH=$JAVA_HOME/bin:$PATH

source ~/.bashrc

java -version

sudo useradd -r -m -s /bin/false nexus

cd /opt
sudo wget https://download.sonatype.com/nexus/3/nexus-unix-x86-64-3.79.0-09.tar.gz

sudo tar -xvzf nexus-unix-x86-64-3.79.0-09.tar.gz

sudo mv nexus-3* nexus

sudo chown -R nexus:nexus /opt/nexus
sudo chown -R nexus:nexus /opt/sonatype-work
sudo chmod -R 750 /opt/nexus
sudo chmod -R 750 /opt/sonatype-work
```

```bash
sudo vi /opt/nexus/bin/nexus.rc
run_as_user="nexus"
```

```bash
sudo vi /etc/systemd/system/nexus.service

[Unit]
Description=Nexus Repository Manager
After=network.target

[Service]
Type=simple
User =nexus
ExecStart=/opt/nexus/bin/nexus run
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl start nexus
sudo systemctl enable nexus
sudo systemctl status nexus
```

http://<your-ec2-public-ip>:8081

Username: admin

```bash
cat /opt/sonatype-work/nexus3/admin.password
```

Set new password may be as admin

And Enable anonymous access

Or

```bash
sudo mkdir -p /nexus-data
sudo chown -R 200 /nexus-data
sudo chmod -R 700 /nexus-data

docker run -d -p 8081:8081 --name nexus  -v /nexus-data:/nexus-data sonatype/nexus3

docker exec and find the password :- docker exec -it nexus cat /nexus-data/admin.password
```

## Snapshot vs Release

Snapshot is for dev purpose it has snapshot as name

And war which doesn't have snapshot in name are release purpose

```xml
    <groupId>com.example</groupId>
    <artifactId>mindcircuitbatch13</artifactId>
    <version>2.246</version>
    <packaging>war</packaging>
<!-- give this to go to release repo -->
```

```xml
    <groupId>com.example</groupId>
    <artifactId>mindcircuitbatch13</artifactId>
    <version>2.246-SNAPSHOT</version>
    <packaging>war</packaging>
<!-- give this to go to snapshot repo -->
```

## To Copy Artifact to Artifactory

Go to setting at the top and go to repositories and create two repo one for release and one for snapshot

Click on create repositories and select maven2(hosted) and give a name and select version policy as release

And create repository 

For snapshots
Click on create repositories and select maven2(hosted) and give a name and select version policy as snapshot

And create repository 

To copy url see the copy button on ui for the selected repo

Go to GitHub repo and edit pom.xml

```xml
	<distributionManagement>
		 <snapshotRepository>
		    <id>NexusRepo</id>
		    <url>http://52.207.250.101:8081/repository/batch14-snapshot/</url>
		 </snapshotRepository>
		
		<repository>
		    <id>NexusRepo</id>
		    <url>http://52.207.250.101:8081/repository/batch14-release/</url>
		</repository>
 	</distributionManagement>
```

Replace the snapshot and release url which copied in nexus site

Now go to Jenkins server vi /etc/maven/settings.xml go to servers 

```xml
    <server>
      <id>deploymentRepo</id>
      <username>repouser</username>
      <password>repopwd</password>
    </server>
    -->
<!-- add this below text in the file -->
    <server>
      <id>NexusRepo</id> <!-- name should be same as pom.xml -->
      <username>admin</username>
      <password>admin</password>
    </server>
```

```xml
    <!-- Another sample, using keys to authenticate.
    <server>
      <id>siteServer</id>
      <privateKey>/path/to/private/key</privateKey>
      <passphrase>optional; leave empty if not used.</passphrase>
    </server>
    -->
  </servers>
```

```bash
use command mvn clean deploy  # to deploy to artifactory
```

Or

Install Config File Provider Plugin and Pipeline Maven Integration Plugin in Jenkins and go to manage Jenkins and managed files and select add a new config and select global maven settings.xml and give a name maven-setting

```xml
    <server>
      <id>deploymentRepo</id>
      <username>repouser</username>
      <password>repopwd</password>
    </server>
    -->
<!-- add this below text in the file -->
    <server>
      <id>NexusRepo</id> <!-- name should be same as pom.xml -->
      <username>admin</username>
      <password>admin</password>
    </server>
```

```xml
    <!-- Another sample, using keys to authenticate.
    <server>
      <id>siteServer</id>
      <privateKey>/path/to/private/key</privateKey>
      <passphrase>optional; leave empty if not used.</passphrase>
    </server>
    -->
  </servers>
```

Edit the above and now go to pipeline syntax and choose withmaven:provide maven environments

Select maven and jdk and global maven settings file path

```groovy
withMaven(globalMavenSettingsConfig: '82e1cd7a-0903-49e3-b706-0b627bdac10d', jdk: 'jdk', maven: 'maven', mavenSettingsConfig: '', traceability: true) {
    sh 'mvn deploy'
}
```

To view the artifact that uploaded to go to browse symbol beside setting and select browse you will find the artifact