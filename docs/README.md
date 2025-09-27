# Mailbook  
**A JSP‑JDBC‑MySQL‑Servlet based Java project to demonstrate CRUD operations where users can create, read, update and delete their name, email and country.**  

---  

## Table of Contents
1. [Overview](#overview)  
2. [Prerequisites](#prerequisites)  
3. [Installation](#installation)  
4. [Configuration](#configuration)  
5. [Running the Application](#running-the-application)  
6. [Usage (Web UI)](#usage-web-ui)  
7. [API Documentation](#api-documentation)  
8. [Examples](#examples)  
9. [Testing](#testing)  
10. [Troubleshooting](#troubleshooting)  
11. [Contributing](#contributing)  
12. [License](#license)  

---  

## Overview
Mailbook is a **pure Java EE** web application that showcases the classic **CRUD** workflow using:

| Layer | Technology |
|-------|------------|
| Presentation | JSP, JSTL, Bootstrap 5 |
| Business / Data Access | Servlets, JDBC |
| Persistence | MySQL |
| Build | Maven (or Gradle) |
| Server | Apache Tomcat 9+ (or any Servlet‑3.1+ container) |

The project is deliberately lightweight – no Spring, no JPA – so you can see the **raw JDBC** calls and servlet routing that power a typical enterprise web app.

---  

## Prerequisites
| Tool | Minimum version | Why |
|------|----------------|-----|
| **JDK** | 11 (or 17) | Language level & `java.sql` APIs |
| **Maven** | 3.6+ (or Gradle 7+) | Build & dependency management |
| **MySQL** | 5.7+ (or MariaDB 10.3+) | Database for persisting users |
| **Tomcat** | 9.0+ (or any Servlet‑3.1+ container) | Deploy the WAR |
| **Git** | any | Clone the repo |
| **Browser** | Chrome/Firefox/Edge | UI testing |

> **Tip:** If you prefer Docker, see the optional Docker section at the bottom of this file.

---  

## Installation  

### 1. Clone the repository
```bash
git clone https://github.com/your‑org/Mailbook.git
cd Mailbook
```

### 2. Set up the MySQL database  

```sql
-- Connect to MySQL as a user with CREATE DATABASE privilege
CREATE DATABASE mailbook CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

USE mailbook;

CREATE TABLE users (
    id      INT AUTO_INCREMENT PRIMARY KEY,
    name    VARCHAR(100) NOT NULL,
    email   VARCHAR(150) NOT NULL UNIQUE,
    country VARCHAR(100) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

> **Optional:** Run the provided `src/main/resources/sql/mailbook-schema.sql` script:
```bash
mysql -u root -p < src/main/resources/sql/mailbook-schema.sql
```

### 3. Configure DB credentials  

Copy the template `src/main/resources/db.properties.example` to `db.properties` and edit the values:

```properties
# src/main/resources/db.properties
jdbc.driverClassName=com.mysql.cj.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/mailbook?useSSL=false&serverTimezone=UTC
jdbc.username=your_mysql_user
jdbc.password=your_mysql_password
```

> **Important:** `db.properties` is **excluded from version control** (`.gitignore`) for security.

### 4. Build the project  

#### Maven
```bash
mvn clean package
```

#### Gradle (if you prefer)
```bash
./gradlew clean build
```

The command produces `target/mailbook-<version>.war` (Maven) or `build/libs/mailbook-<version>.war` (Gradle).

---  

## Configuration  

| File | Purpose | Typical changes |
|------|---------|-----------------|
| `src/main/webapp/WEB-INF/web.xml` | Servlet mappings, welcome file | Add filters, change welcome page |
| `src/main/resources/db.properties` | JDBC connection details | DB host, user, password |
| `src/main/resources/log4j2.xml` | Logging configuration | Log level, file appender |
| `src/main/webapp/WEB-INF/jsp/*.jsp` | UI templates | Branding, CSS/JS includes |

If you need a **different table name** or extra columns, edit `src/main/java/com/mailbook/dao/UserDao.java` and the SQL statements accordingly.

---  

## Running the Application  

### Deploy to Tomcat (manual)

1. Copy the WAR to Tomcat’s `webapps` directory:
   ```bash
   cp target/mailbook-*.war $CATALINA_HOME/webapps/mailbook.war
   ```
2. Start (or restart) Tomcat:
   ```bash
   $CATALINA_HOME/bin/startup.sh
   ```
3. Open a browser and navigate to:  
   ```
   http://localhost:8080/mailbook/
   ```

### Run with Embedded Jetty (quick test)

```bash
mvn jetty:run
# or
./gradlew jettyRun
```

The app will be reachable at `http://localhost:8080/`.

---  

## Usage – Web UI  

| Action | How to do it |
|--------|--------------|
| **List all users** | Home page (`/mailbook/`) shows a table of all records. |
| **Create a new user** | Click **Add New** → fill the form → **Save**. |
| **Edit a user** | Click the **Edit** icon next to a row → modify fields → **Update**. |
| **Delete a user** | Click the **Trash** icon → confirm the JavaScript prompt. |
| **Search** | Use the search box on the list page (simple `LIKE` query). |

All UI pages are built with **Bootstrap 5** for responsive design and **JSTL** for server‑side rendering.

---  

## API Documentation  

The application also exposes a **REST‑like** API (implemented as servlets) for programmatic access. All endpoints return **JSON** and accept **application/json** payloads.

| Method | URL | Description | Request Body | Success Response |
|--------|-----|-------------|--------------|------------------|
| `GET` | `/mailbook/api/users` | Retrieve all users | – | `200 OK` → `[{id, name, email, country, created_at}, …]` |
| `GET` | `/mailbook/api/users/{id}` | Retrieve a single user | – | `200 OK` → `{id, name, email, country, created_at}` |
| `POST` | `/mailbook/api/users` | Create a new user | `{ "name":"...", "email":"...", "country":"..." }` | `201 Created` → `{ "id": 12, ... }` |
| `PUT` | `/mailbook/api/users/{id}` | Update an existing user | `{ "name":"...", "email":"...", "country":"..." }` | `200 OK` → updated object |
| `DELETE` | `/mailbook/api/users/{id}` | Delete a user | – | `204 No Content` |

### Common Headers
```http
Content-Type: application/json
Accept: application/json
```

### Error Responses
| Code | Meaning | Example Body |
|------|---------|--------------|
| `400` | Bad request / validation error | `{ "error":"Email already exists" }` |
| `404` | Resource not found | `{ "error":"User with id 99 not found" }` |
| `500` | Server error (SQL, etc.) | `{ "error":"Internal server error" }` |

### Servlet Mapping (web.xml excerpt)

```xml
<servlet>
    <servlet-name>UserApiServlet</servlet-name>
    <servlet-class>com.mailbook.api.UserApiServlet</servlet-class>
</servlet>

<servlet-mapping>
    <servlet-name>UserApiServlet</servlet-name>
    <url-pattern>/api/users/*</url-pattern>
</servlet-mapping>
```

---  

## Examples  

### 1. Create a user (cURL)

```bash
curl -X POST http://localhost:8080/mailbook/api/users \
     -H "Content-Type: application/json" \
     -d '{
           "name": "Alice Johnson",
           "email": "alice@example.com",
           "country": "Canada"
         }'
```

**Response**

```json
{
  "id": 23,
  "name": "Alice Johnson",
  "email": "alice@example.com",
  "country": "Canada",
  "created_at": "2025-09-27T14:32:10Z"
}
```

### 2. Get all users

```bash
curl http://localhost:8080/mail