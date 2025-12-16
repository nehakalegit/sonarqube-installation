Manual way to install SonarQube with PostgreSQL** on a **Amazon Linux EC2 instance**. Sonarqube to run requires atleast 2GB RAM - t3.medium

## **STEP 1: Update system**

sudo dnf update -y

## **STEP 2: Install Java 17**

sudo dnf search java-17

sudo dnf install java-17

sudo java --version

## **STEP 3: Install PostgreSQL**

// postgresql-server and postgresql these are two separate packages. Postgresql - Client tools only-psql (CLI client), pg_dump, pg_restore, createdb, dropdb, Client libraries

//Postgresql-server - The actual database server, postgres daemon(for setting username and password), initdb, Systemd service files, Default data directory setup

//Even though sonarqube server have default database,it is not used in real production environment. New database is created

//When postgresql is installed, one user 'postgres' is creasted and postgresql-setup in /usr/bin

sudo dnf search postgresql 

sudo dnf install postgresql15 postgresql15-server -y

sudo /usr/bin/postgresql-setup initdb  //PostgreSQL database initialization

sudo systemctl start postgresql

sudo systemctl enable postgresql

## **STEP 4: Create SonarQube database & user**

sudo cat /etc/passwd|tail -n 5  // One user is added 'postgres' during postgresql installation

sudo su - postgres //Switch to user. Owns the PostgreSQL data directory (/var/lib/pgsql/data),Runs the PostgreSQL server process (postgres daemon),Used to create database                          roles, databases, and manage PostgreSQL

psql   //psql is like the terminal interface to PostgreSQL

CREATE DATABASE sonarqube;

CREATE USER sonar WITH ENCRYPTED PASSWORD 'Sonar@123';    //Creates a new database user (also called a role) named sonar.

GRANT ALL PRIVILEGES ON DATABASE sonarqube TO sonar;    //Gives the sonar user full access to the sonarqube database.

\q    //Exits the psql interactive shell.

exit

## **STEP 5: Download & install SonarQube**

cd /opt

sudo yum install wget unzip -y

sudo wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.9.2.77730.zip

sudo unzip sonarqube-9.9.2.77730.zip

sudo mv sonarqube-9.9.2.77730 sonarqube

## **STEP 6: Create SonarQube user**

sudo useradd sonar      // This is a system/Linux user,SonarQube is a Java application that runs on Linux. We do not want it to run as root for security reasons.The sonar                                Linux user will own the SonarQube installation files and run the SonarQube process.

sudo chown -R sonar:sonar /opt/sonarqube


## **STEP 7: Configure SonarQube for PostgreSQL**

sudo vim /opt/sonarqube/conf/sonar.properties


Add and uncomment:

sonar.jdbc.username=sonar

sonar.jdbc.password=Sonar@123

sonar.jdbc.url=jdbc:postgresql://localhost/sonarqube

sonar.web.host=0.0.0.0


## **STEP 8: Configure system limits**

//SonarQube is not just a simple Java app. Internally it runs:Elasticsearch (for indexing & search), Web server,Compute engine,Multiple threads + many open files.Linux has default limits that are too low for this workload. Because SonarQube embeds Elasticsearch, which requires higher memory mappings, file descriptors, and thread limits than default Linux values. Without tuning kernel and user limits, SonarQube becomes unstable or fails to start.

sudo vim /etc/sysctl.conf  //This file controls kernel (OS) limits for the whole machine.A memory map is how Linux lets programs access memory efficiently.

Add:
vm.max_map_count=262144  //vm-virtual machine. Maximum number of memory-mapped areas a process can have. Elasticsearch uses memory-mapped files heavily. By default it is 65530

fs.file-max=65536    //Maximum number of files the system can have open at once(file descriptors).SonarQube + PostgreSQL + Java open many files:logs, indexes, temp files

Apply changes:

sudo sysctl -p

sudo vim /etc/security/limits.conf   //This file controls how much resources each linux user can use.SonarQube runs as the Linux user sonar. NOT as root. NOT as postgres

Add:

sonar   -   nofile   65536 //Max number of open files per process for the sonar Linux user. SonarQube runs as the sonar user. Default limit is often 1024,Not enough for                                    indexing + analysis

sonar   -   nproc    4096  //Max number of processes/threads the sonar user can create. SonarQube runs:web threads,background workers,Elasticsearch threads

## **STEP 9: Start SonarQube**

sudo su - sonar

/opt/sonarqube/bin/linux-x86-64/sonar.sh start

Check status:

/opt/sonarqube/bin/linux-x86-64/sonar.sh status


## **STEP 🔟 Open firewall**

* Allow **TCP 9000** in **EC2 Security Group**

## **STEP 1️⃣1️⃣ Access SonarQube**

http://EC2_PUBLIC_IP:9000

**Login:**

* Username: `admin`
  
* Password: `admin`
  
  (Change password immediately)

