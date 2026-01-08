# 3-Tier Application Deployment using Docker
This repository documents the setup and deployment of a **3-tier web application** using **Docker**, based on the handwritten notes provided.

---

## 🧰 Tech Stack

* **Frontend:** Node.js, npm, Apache
* **Backend:** Java 17, Maven, Spring Boot
* **Database:** MariaDB (MySQL)
* **Containerization:** Docker

```
Database (RDS) --> Backend --> Frontend
```
---

## 🗄️ Database Setup (AWS RDS)

1. Go to **Amazon RDS → Create database**.
2. Choose:
   - Engine: **MariaDB**
   - Template: **Free tier**
   - Username & Password (example: password `Redhat`, but choose your own secure password)
   - Storage: `gp2`
   - Enable automatic backups (recommended)
3. Configure **Security Group** (would be the same as your EC2):
   - Allow **inbound** traffic on port **3306** from your EC2 instance’s security group.
4. Create the database instance.
5. Note down the **RDS endpoint** – this will become your `DB_HOST`.

You will finally have environment info like:

```txt
DB_HOST = <rds-endpoint>
DB_USER = <db-username>
DB_PASS = <db-password>
DB_NAME = student_db
```
---

## ☁️ EC2 Instance Setup

1. Launch EC2 (Amazon Linux / Ubuntu)
2. Configure:

   * Key pair
   * Same security group with RDS
   * Open required ports (80, 8080)

### Connect to RDS from EC2
Launch an EC2 and use following commands :
```bash
sudo apt update
sudo apt install mysql-client -y
mysql -h <RDS-ENDPOINT> -u <USERNAME> -p
```

Inside MySQL:

```sql
CREATE DATABASE student_db;
GRANT ALL PRIVILEGES ON springbackend.* TO 'username'@'localhost' IDENTIFIED BY 'your_password';

USE student_db;

CREATE TABLE `students` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `name` varchar(255) DEFAULT NULL,
  `email` varchar(255) DEFAULT NULL,
  `course` varchar(255) DEFAULT NULL,
  `student_class` varchar(255) DEFAULT NULL,
  `percentage` double DEFAULT NULL,
  `branch` varchar(255) DEFAULT NULL,
  `mobile_number` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=80 DEFAULT CHARSET=latin1 COLLATE=latin1_swedish_ci;

EXIT;
```
---
Install Docker:

```bash
sudo apt install docker.io -y
git clone <Link to Clone EasyCRUD Repo>
```

---

## 🧩 Backend Docker Setup (Spring Boot)

### Recommended Base Image

```bash
cd EasyCRUD/backend/
cp src/main/resources/application.properties .
nano application.properties 
```
In application.properties make the following changes:

```bash
vim backend/src/main/resources/application.properties

   server.port=8080
   spring.datasource.url=jdbc:mariadb://<DB_HOST>:3306/<DB_NAME>
   spring.datasource.username=<DB_USER>
   spring.datasource.password=<DB_PASS>
```

Use a predefined Maven + Java image:

```dockerfile
FROM maven:3.8.3-openjdk-17 

COPY . /opt/

WORKDIR /opt

RUN rm -rf src/main/resources/application.properties
RUN cp app.prop src/main/resources/application.properties

RUN mvn clean package

WORKDIR target

EXPOSE 8080

CMD ["java","-jar","student-regis-0.0.1-SNAPSHOT.jar"]
```

> 📌 Use the JAR name from your `pom.xml` (`artifactId`).

### Build & Run Backend Container

```bash
docker build -t backend:v1 .
docker run -d -p 8080:8080 backend:v1
docker ps
```

✅ Backend application is now portable and running in a container.

---

## 🎨 Frontend Docker Setup

### Environment Configuration

Update `.env` file with Backend **Public IP**:

```
REACT_APP_BACKEND_URL=http://<EC2-PUBLIC-IP>:8080
```

---

### Frontend Dockerfile

```dockerfile
FROM node:25-alpine

WORKDIR /opt

COPY . /opt/

RUN npm install
RUN npm run build

RUN apk update
RUN apk add apache2

RUN cp -rf dist/* /var/www/localhost/htdocs/

EXPOSE 80

ENTRYPOINT ["httpd","-D","FOREGROUND"]
```

### Build & Run Frontend Container

```bash
docker build -t frontend:v1 .
docker run -d -p 80:80 frontend:v1
docker ps
```

---

## 🧹 Maintenance Notes

* Always remove old images before rebuilding:

```bash
docker images
docker rmi <image-id>
```

* Stop unused containers:

```bash
docker ps
docker stop <container-id>
```

---

## ✅ Final Result

* Frontend accessible via **EC2 Public IP** on port **80**
* Backend running on port **8080**
* Database managed securely via **AWS RDS**

🎉 Your **3-tier application is fully containerized and portable!**

---

## 📌 Notes

* Keep backend and frontend in separate Dockerfiles
* Prefer prebuilt images (Maven, Node Alpine) for efficiency
* Use `/opt` as best practice working directory inside containers
