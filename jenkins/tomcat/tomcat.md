# Tomcat Installation

First install java `sudo yum install java -y`

First cd to /opt

Go to the official tomcat download page

Copy core:tar.gz link use right click and copy link address

```bash
wget https://dlcdn.apache.org/tomcat/tomcat-11/v11.0.2/bin/apache-tomcat-11.0.2.tar.gz
```

Apache-tomcat folder will be created

```bash
tar -xvf apache-tomcat-11.0.2.tar.gz
```

## Configuration

You can cd into apache-tomcat-10.1.33 and cd into /opt/apache-tomcat-10.1.33/conf and edit the server.xml so you can run Jenkins and tomcat on one server edit connector port

And edit port to connect the tomcat to another port

We should also define user for tomcat

Edit tomcat-user.xml in the same directory and delete everything and paste the below

```xml
<tomcat-users>
    <!-- Define a user with all roles -->
    <role rolename="admin-gui"/>
    <role rolename="manager-gui"/>
    <role rolename="manager-script"/>
    <role rolename="admin-script"/>
    <user username="admin" password="pass" roles="admin-gui,manager-gui,manager-script,admin-script"/>
</tomcat-users>
```

Now cd to /opt/apache-tomcat-10.1.33/webapps/manager/META-INF and edit context.xml and delete the allow="delete this" section and add .*

Now cd to /opt/apache-tomcat-10.1.33/bin and run startup.sh script `./startup.sh` to run the tomcat server

## War Files

Webapps is the folder where the war files will be copied and run

## Deploy to Tomcat Through Jenkins (Plugin)

We can automate the deployment process by installing a plugin called deploy to container

And in build go to post build actions select deploy war to container

Give war file name or **/*.war and context-path(name)

Now select add containers select latest or the version installed and give the user credentials login and pass of user and tomcat url

## Automated Script to Install Tomcat

Make sure you open the container port

```bash
#!/bin/bash
cd /opt
sudo wget https://dlcdn.apache.org/tomcat/tomcat-11/v11.0.2/bin/apache-tomcat-11.0.2.tar.gz 
sudo tar -xvf apache-tomcat-11.0.2.tar.gz
cd /opt/apache-tomcat-11.0.2/webapps/manager/META-INF
sudo sed -i 's/"127\\\\.\\\\d+\\\\.\\\\d+\\\\.\\\\d+|::1|0:0:0:0:0:0:0:1"/".*"/g' context.xml
cd /opt/apache-tomcat-11.0.2/conf
sudo mv tomcat-users.xml tomcat-users_bkup.xml
sudo touch tomcat-users.xml
sudo echo '<?xml version="1.0" encoding="utf-8"?>
        <tomcat-users>
        <role rolename="manager-gui"/>
        <user username="admin" password="pass" roles="manager-gui, manager-script, manager-status"/>
        </tomcat-users>' > tomcat-users.xml

cd /opt/apache-tomcat-11.0.2/conf/
sudo sed -i 's/Connector port="8080"/Connector port="8081"/g' server.xml
sudo /opt/apache-tomcat-11.0.2/bin/startup.sh
```

You can go to the GitHub repo folder and run mvn clean install to build a war file

## Running Tomcat with Docker Container

Dockerfile for tomcat to run on root this will make website directly access on ip:8080

```dockerfile
# Use an official Tomcat base image
FROM tomcat:10.1-jdk17

# Set environment variables (optional)
ENV CATALINA_HOME /usr/local/tomcat
ENV PATH $CATALINA_HOME/bin:$PATH

# Remove the default webapps for a clean slate (optional)
RUN rm -rf /usr/local/tomcat/webapps/*

# Copy your WAR file into the webapps directory
COPY mindcircuitbatch13-2.246-SNAPSHOT.war /usr/local/tomcat/webapps/ROOT.war

# Expose the default Tomcat port
EXPOSE 8080

# Start Tomcat
CMD ["catalina.sh", "run"]
```

## Multi Stage Docker Container

```dockerfile
FROM maven AS buildstage
RUN mkdir /opt/webpage
WORKDIR /opt/webpage
COPY . .
RUN mvn clean install 

FROM tomcat
WORKDIR webapps
COPY --from=buildstage /opt/webpage/target/*.war .
RUN rm -rf ROOT && mv *.war ROOT.war
EXPOSE 8080
```