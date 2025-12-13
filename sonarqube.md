Got it! Here’s the **easiest manual way to install SonarQube with PostgreSQL** on a **Red Hat (RHEL) EC2 instance**. I’ll keep it **step-by-step and minimal**.

---

# ✅ **SonarQube + PostgreSQL on Red Hat EC2**

---

## **STEP 1: Update system**

```bash
sudo yum update -y
```

---

## **STEP 2: Install Java 17**

```bash
sudo yum install java-17-openjdk -y
java -version
```

---

## **STEP 3: Install PostgreSQL**

```bash
sudo yum install postgresql-server postgresql-contrib -y
sudo postgresql-setup initdb
sudo systemctl start postgresql
sudo systemctl enable postgresql
```

---

## **STEP 4: Create SonarQube database & user**

```bash
sudo su - postgres
psql
```

```sql
CREATE DATABASE sonarqube;
CREATE USER sonar WITH ENCRYPTED PASSWORD 'StrongPassword';
GRANT ALL PRIVILEGES ON DATABASE sonarqube TO sonar;
\q
exit
```

---

## **STEP 5: Download & install SonarQube**

```bash
cd /opt
sudo yum install wget unzip -y
sudo wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.9.2.77730.zip
sudo unzip sonarqube-9.9.2.77730.zip
sudo mv sonarqube-9.9.2.77730 sonarqube
```

---

## **STEP 6: Create SonarQube user**

```bash
sudo useradd sonar
sudo chown -R sonar:sonar /opt/sonarqube
```

---

## **STEP 7: Configure SonarQube for PostgreSQL**

```bash
sudo nano /opt/sonarqube/conf/sonar.properties
```

Add:

```ini
sonar.jdbc.username=sonar
sonar.jdbc.password=StrongPassword
sonar.jdbc.url=jdbc:postgresql://localhost/sonarqube
```

---

## **STEP 8: Configure system limits**

```bash
sudo nano /etc/sysctl.conf
```

Add:

```conf
vm.max_map_count=262144
fs.file-max=65536
```

Apply changes:

```bash
sudo sysctl -p
```

```bash
sudo nano /etc/security/limits.conf
```

Add:

```conf
sonar   -   nofile   65536
sonar   -   nproc    4096
```

---

## **STEP 9: Start SonarQube**

```bash
sudo su - sonar
/opt/sonarqube/bin/linux-x86-64/sonar.sh start
```

Check status:

```bash
/opt/sonarqube/bin/linux-x86-64/sonar.sh status
```

---

## **STEP 🔟 Open firewall**

* Allow **TCP 9000** in **EC2 Security Group**

---

## **STEP 1️⃣1️⃣ Access SonarQube**

```
http://EC2_PUBLIC_IP:9000
```

**Login:**

* Username: `admin`
* Password: `admin`
  (Change password immediately)

---

## ✅ Notes

* Minimum **2GB RAM**, better **4GB** for production
* Don’t run SonarQube as root
* For production: use **systemd service + SSL (Nginx/Apache)**

---

If you want, I can give you a **super-easy one-liner setup script** for Red Hat EC2 that **installs Java, PostgreSQL, and SonarQube automatically**.

Do you want me to create that?
