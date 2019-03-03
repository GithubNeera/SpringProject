## Create Amazon EC2 Instance

* Ensure that inbound `MySQL` port `3306` is opened
* `chmod 700 "keypair-name.pem"`

## Install MySQL

To install MySQL, run the following command from a terminal prompt:
* `sudo apt-get install mysql-server`

During the installation process you will be prompted to enter a password for the MySQL root user.
* `username / password : root / root`

Once the installation is complete, the MySQL server should be started automatically. You can run the following command from a terminal prompt to check whether the MySQL server is running:
* `sudo netstat -tap | grep mysql`

When you run this command, you should see the following line or something similar:
* `tcp        0      0 localhost:mysql         *:*                     LISTEN      2759/mysqld`

If the server is not running correctly, you can type the following command to start it:
* `sudo service mysql restart`

If you would like to change the MySQL root password, in a terminal enter:
* `sudo dpkg-reconfigure mysql-server-5.5`

The MySQL daemon will be stopped, and you will be prompted to enter a new password.

Create an empty schema and a sonarqube user. Grant this sonarqube user permissions to create, update and delete objects for this schema. The charset of the database has to be set to "UTF-8" and the language (database and user) to "English". 

* Here is a [MYSQL Script](https://github.com/SonarSource/sonar-examples/blob/master/scripts/database/mysql/create_database.sql)

```bash
vi create_database.sql
Create SonarQube database and user.
Command: mysql -u root -p < create_database.sql
```

```bash
CREATE DATABASE sonar CHARACTER SET utf8 COLLATE utf8_general_ci;
CREATE USER ‘sonar’ IDENTIFIED BY ‘sonar';
GRANT ALL ON sonar.* TO ‘sonar’@’%’ IDENTIFIED BY ‘sonar';
GRANT ALL ON sonar.* TO ‘sonar’@’localhost’ IDENTIFIED BY ‘sonar';
FLUSH PRIVILEGES;
```

* To run the script: `sudo mysql -u root -p < create_database.sql`
* To verify the user has been created: `SELECT User FROM mysql.user;` you should see user with name `sonar`
* To verify the database has been created: `show databases;` you should see database with name `sonar`

* Modify my.cnf
```bash
sudo nano /etc/mysql/my.cnf 

find the line:
bind-address            = 127.0.0.1

and change it to:
bind-address            = 0.0.0.0
```
then restart mysql

## Install Java

```bash
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get install oracle-java7-installer
```

## Install Sonar

##### Install
* Add the following entry in your /etc/apt/sources.list: `cd /etc/apt/` then `sudo vim sources.list` then `press o` then copy 
`deb http://downloads.sourceforge.net/project/sonar-pkg/deb binary/` then `press ESC` then write `:wq!` then `press ENTER`
* `sudo apt-get update`
* `sudo apt-get install sonar`

##### Configure Database
* Go to `/opt/sonar/conf`
* `sudo nano sonar.properties`
* Comment the embedded database and uncomment the line to use MySQL

```bash
sudo vi /opt/sonar$ sudo vi conf/sonar.properties

#sonar.jdbc.url=jdbc:h2:tcp://localhost:9092/sonar

#—– MySQL 5.x
# Comment the embedded database and uncomment the following line to use MySQL
sonar.jdbc.url=jdbc:mysql://localhost:3306/sonar?useUnicode=true&characterEncoding=utf8&rewriteBatchedStatements=true
```

##### Configure WebServer
* `sonar.web.context=/sonar`
* `sonar.web.port=9000`

##### Start
* `cd /opt/sonar/bin` `sudo sh sonar.sh start` `sudo sh sonar.sh stop`
* `cd /opt/sonar/log` `tail -f sonar.log`
* `ps -ef  | grep sonar`

## Analyzing with Maven

* Adding following in your `.m2/settings.xml`
```bash
<settings>
  <profiles>
    <profile>
        <id>sonar</id>
        <activation>
            <activeByDefault>true</activeByDefault>
        </activation>
        <properties>
            <sonar.jdbc.url>
              jdbc:mysql://www.myserver.com:3306/sonar?useUnicode=true&amp;characterEncoding=utf8&amp;rewriteBatchedStatements=true&amp;useConfigs=maxPerformance
            </sonar.jdbc.url>
            <sonar.jdbc.username>sonar</sonar.jdbc.username>
            <sonar.jdbc.password>sonar</sonar.jdbc.password>
          
            <sonar.host.url>
              http://www.myserver.com/sonar/
            </sonar.host.url>
        </properties>
    </profile>
 </profiles>
</settings>
```
* Add following `sonar-maven-plugin` in the project `pom` file
```bash
            <!--
                SonarQube™ is an open source platform for Continuous Inspection of code quality.
                The Maven Plugin triggers the code analyzers.
            -->
            <plugin>
                <groupId>org.codehaus.mojo</groupId>
                <artifactId>sonar-maven-plugin</artifactId>
                <version>2.4</version>
            </plugin>
```
* Run these commands
```bash
mvn clean install
mvn sonar:sonar
```
* Now go to your sonar and you will find your project!

## Configure SonarQube in IntelliJ

http://docs.codehaus.org/display/SONAR/SonarQube+in+IntelliJ

## Configure SonarQube in Eclipse

http://docs.codehaus.org/display/SONAR/SonarQube+in+Eclipse