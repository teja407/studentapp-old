
ubuntu 18.05

sudo apt update -y
sudo apt install tomcat8 -y
java -version
sudo apt-get install openjdk-8-jdk -y
sudo update-alternatives --config java
java -version
git clone https://github.com/teja407/studentapp-old.git
sudo apt install maven -y
mvn clean package -DskipTests
sudo cp /home/azadmin/studentapp-old/target/studentapp-2.7-SNAPSHOT.war /var/lib/tomcat8/webapps/student.war
sudo su -
cd /var/lib/tomcat8/webapps/
<IP>:8080/student
sudo apt install mariadb-server -y
systemctl enable mariadb

# mysql

> create database studentapp;
> use studentapp;
> CREATE TABLE Students(student_id INT NOT NULL AUTO_INCREMENT,
	student_name VARCHAR(100) NOT NULL,
  student_addr VARCHAR(100) NOT NULL,
	student_age VARCHAR(3) NOT NULL,
	student_qual VARCHAR(20) NOT NULL,
	student_percent VARCHAR(10) NOT NULL,
	student_year_passed VARCHAR(10) NOT NULL,
	PRIMARY KEY (student_id)
);
> grant all privileges on studentapp.* to 'student'@'%' identified by 'student@1';
> flush privileges;

cd /var/lib/tomcat8/
wget https://github.com/cit-ager/APP-STACK/raw/master/mysql-connector-java-5.1.40.jar -O lib/mysql-connector-java-5.1.40.jar
vim conf/context.xml

<Resource name="jdbc/TestDB" auth="Container" type="javax.sql.DataSource"
               maxActive="50" maxIdle="30" maxWait="10000"
               username="student" password="student@1"
               driverClassName="com.mysql.jdbc.Driver"
               url="jdbc:mysql://localhost:3306/studentapp"/>

systemctl restart tomcat8

check the service - http://<IP-Address>:8080/student/

========================

webserver - install

sudo apt update
sudo apt install apache2 -y
sudo systemctl status apache2

vi /etc/apache2/workers.properties

# Define 1 real worker using ajp13 
worker.list=worker1 
# Set properties for worker (ajp13) 
worker.worker1.type=ajp13
worker.worker1.host=localhost
worker.worker1.port=8009 

sudo apt-get install libapache2-mod-jk -y
cat /etc/apache2/mods-enabled/jk.load
rm -rf /etc/apache2/mods-enabled/jk.conf
vi /etc/apache2/mods-enabled/jk.conf

<IfModule jk_module>
# Where to find workers.properties
# Update this path to match your conf directory location
JkWorkersFile /etc/apache2/workers.properties
# Where to put jk logs
# Update this path to match your logs directory location
JkLogFile /var/log/apache2/mod_jk.log
# Set the jk log level [debug/error/info]
JkLogLevel info
# Select the log format
JkLogStampFormat "[%a %b %d %H:%M:%S %Y]"
# JkOptions indicate to send SSL KEY SIZE,
JkOptions +ForwardKeySize +ForwardURICompat -ForwardDirectories
# JkRequestLogFormat set the request format
JkRequestLogFormat "%w %V %T"
# Shm log file
JkShmFile /var/log/apache2/jk-runtime-status
</IfModule>

vi /etc/apache2/sites-available/000-default.conf
JkMount /student* worker1

systemctl restart apache2

tail -f /var/log/apache2/access.log
tail -f /var/log/apache2/mod_jk.log
vi /var/lib/tomcat8/conf/server.xml
port 8009 open
systemctl restart tomcat8


http://<IP>/student/ - open
