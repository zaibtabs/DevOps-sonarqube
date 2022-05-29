
# Sonarqube Setup

SonarQube is an open-source static testing analysis software, it is used by developers to manage source code quality and consistency.
## 🧰 Prerequisites
1. Centos 7.0 on VMDesktop
2. Install Java-11
  ```sh 
    yum get update   
     list | grep openjdk-11  
   $ sudo update-alternatives --config 'java'
   $ sudo yum install java-11-openjdk
   
   $ java -version

openjdk version "11.0.14" 2022-01-18 LTS LTS
OpenJDK Runtime Environment 18.9 (build 11.0.14+9-LTS)
OpenJDK 64-Bit Server VM 18.9 (build 11.0.14+9-LTS, mixed mode, sharing)

$ sudo update-alternatives --config 'java'
   ```

## Install & Setup Postgres Database for SonarQube
`Source: (https://www.postgresql.org/download/linux/redhat/)
1. Install Postgres database   
  ```sh 
  sudo yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
sudo yum install -y postgresql14-server
sudo /usr/pgsql-14/bin/postgresql-14-setup initdb
sudo systemctl enable postgresql-14
sudo systemctl start postgresql-14
  ```

2. Set a password and connect to database (setting password as "admin" password)
  ```sh 
  sudo passwd postgres
  su - postgres
  ```

3. Create a database user and database for sonarque 
  ```sh 
  createuser sonar
  psql
  ALTER USER sonar WITH ENCRYPTED PASSWORD 'admin';
  CREATE DATABASE sonarqube OWNER sonar;
  GRANT ALL PRIVILEGES ON DATABASE sonarqube to sonar;
  ``` 

4. Restart postgres database to take latest changes effect 
  ```sh 
  systemctl restart  postgresql-14
  systemctl status  postgresql-14
  ```
`check point`: You should see postgres is running on 5432

yum install net-tools

Go to this ulr and you can see the requirements: https://docs.sonarqube.org/latest/requirements/requirements/`

5. Added below entries in `/etc/sysctl.conf`
  ```sh 
  vm.max_map_count=524288
  fs.file-max=131072
  ulimit -n 131072
  ulimit -u 8192
  ```
6. Add below entries in `/etc/security/limits.conf`
  ```sh 
  sonarqube   -   nofile   131072
  sonarqube   -   nproc    8192
  ```

7. reboot the server 
  ```sh 
  init 6
  ```
  ==========================================

 ## SonarQube Setup

1. Download [soarnqube](https://www.sonarqube.org/downloads/) and extract it.   
  ```sh 
  cd /opt
  wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-8.9.8.54436.zip
  unzip sonarqube-8.9.2.46101.zip
  ```

2. Update sonar.properties with below information 
  ```sh
  sonar.jdbc.username=<sonar_database_username>
  sonar.jdbc.password=<sonar_database_password>

  #sonar.jdbc.username=sonar  # uncomment
  #sonar.jdbc.password=admin  #uncomment
  #sonar.jdbc.url=jdbc:postgresql://localhost/sonarqube   #uncomment and update as seen here
  #sonar.search.javaOpts=-Xmx512m -Xms512m -XX:MaxDirectMemorySize=256m -XX:+HeapDumpOnOutOfMemoryError  #uncommet this line
  ``

3. Create a `/etc/systemd/system/sonarqube.service` file start sonarqube service at the boot time 
  ```sh   
  cat >> /etc/systemd/system/sonarqube.service <<EOL
  [Unit]
  Description=SonarQube service
  After=syslog.target network.target

  [Service]
  Type=forking
  User=sonar
  Group=sonar
  PermissionsStartOnly=true
  ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start 
  ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop
  StandardOutput=syslog
  LimitNOFILE=65536
  LimitNPROC=4096
  TimeoutStartSec=5
  Restart=always

  [Install]
  WantedBy=multi-user.target
  EOL
  ```
4. Important:  in the above file we are using sonarqube as a directory
   ---sh
   mv sonarqube-8.9.8.54436/ sonarqube
   ---
5. Add sonar user and grant ownership to /opt/sonarqube directory 
  ```sh 
  useradd -d /opt/sonarqube sonar
  chown -R sonar:sonar
  ```

6. Reload the demon and start sonarqube service 
  ```sh 
  systemctl daemon-reload 
  systemctl start sonarqube.service 
  ```


## 🧹 CleanUp  

   Stop sonarqube services and delete the instance

 ## Unable to access Sonarqube from browser? 

 1. Make sure port 9000 is opened at security group leave
 2. start sonar service as a sonar user 
 3. user correct database credentials in the sonar.properties
 4. use instance which has atleast 2 GB of RAM
 

   
## 🔗 My Profile

[![linkedin](https://img.shields.io/badge/linkedin-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/salmansayeed/)

  ### 💡 Help/Suggestions or 🐛 Bugs

Thank you for your interest in contributing to our project. Whether it is a bug report, new feature, correction, or additional documentation or solutions, we greatly value feedback and contributions from our community.
