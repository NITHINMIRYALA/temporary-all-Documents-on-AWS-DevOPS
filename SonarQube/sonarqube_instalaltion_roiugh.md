Sonarqube Installation: 
=======================
Prerequisites:
==============
1) EC2 instance with Java installed
2)Use t2.large with atleast 20gb memory to run sonarqube.
3) MySQL Database Server or MyQL RDS instance. 

# Install JAVA
yum install java-1.8*

# Add mysql rpm Repository
yum localinstall https://dev.mysql.com/get/mysql57-community-release-el7-9.noarch.rpm

# Install MySQL 5.7
yum -y install mysql-community-server

#Start MySQL and Enable Start at Boot Time
systemctl start mysqld
systemctl enable mysqld

#Check if mysql is running or not
netstat -na | grep 3306

#Configure the MySQL Root Password 
#You will see default MySQL root password
grep 'temporary' /var/log/mysqld.log

#Login to mysql using the default password
mysql -u root -p

#Now replace the default password with a new and strong password
ALTER USER 'root'@'localhost' IDENTIFIED BY 'Admin@123';
flush privileges;

# Testing using new password
mysql -u root -p

#Download & unzip SonarQube 6.0
cd /opt
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-6.6.zip
unzip sonarqube-6.6.zip
mv /opt/sonarqube-6.6 /opt/sonar

#Login to mysql and create new sonar database
mysql -u root -p
CREATE DATABASE sonar CHARACTER SET utf8 COLLATE utf8_general_ci;

# Create a local and a remote user
CREATE USER sonar@localhost IDENTIFIED BY 'Sonar@123';
CREATE USER sonar@'%' IDENTIFIED BY 'Sonar@123';

#Grant database access permissions to users
GRANT ALL ON sonar.* TO sonar@localhost;
GRANT ALL ON sonar.* TO sonar@'%';

# check users and databases
show databases;
SELECT User FROM mysql.user;
FLUSH PRIVILEGES;
QUIT

#ON EC2 Instance: Edit sonar properties file to uncomment and provide required information for below properties
File Name: /opt/sonar/conf/sonar.properties
sonar.jdbc.username=sonar
sonar.jdbc.password=Sonar@123
sonar.jdbc.url=jdbc:mysql://localhost:3306/sonar?useUnicode=true&characterEncoding=utf8&rewriteBatchedStatements=true&useConfigs=maxPerformance&useSSL=false
sonar.web.host=0.0.0.0
sonar.web.context=/sonar

# Execute the below command if elastic search is not starting
sysctl -w vm.max_map_count=65536

#Sonar version 7 and higher cannot be run using root user,
#please switch to any other user and 
#change the permissions to sonar directory and start sonar.

# Create new user
useradd sabear
passwd sabear

#Add user sabear to sudoers file
vi /etc/sudoers
``
## Same thing without a password
# %wheel        ALL=(ALL)       NOPASSWD: ALL
sabear          ALL=(ALL)       NOPASSWD: ALL
```
# Restart the sshd service
systemctl restart sshd

#Start SonarQube service
cd /opt/sonar/bin/linux-x86-64/
./sonar.sh start

#SonarQube application uses port 9000. access SonarQube from browser

http://<EC2_PUBLIC_IP>:9000/sonar

Note:
=====
1) Check whether you enabled port 9000 in EC2 instance security group

Important Points:
=================
1) mysql port number : 3306
2) sonarqube port: 9000
3) sonarqube logs : /opt/sonar/logs
4) We can find four different logs
   a)access.log
   b)es.log aka elastic search
   c)sonar.log
   d)web.log
   
 
Videos for reference: 
sonarqube installation: https://www.youtube.com/watch?v=zRQrcAi9UdU
mysql rds instance aws: https://www.youtube.com/watch?v=vLaW6b441x0
