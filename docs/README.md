# Mailbook  
**A JSP‑JDBC‑MySQL‑Servlet based Java project to demonstrate CRUD operations where users can create, update, edit and delete their name, email and country respectively.**  

---  

## Table of Contents
1. [Prerequisites](#prerequisites)  
2. [Installation](#installation)  
3. [Configuration](#configuration)  
4. [Running the Application](#running-the-application)  
5. [Usage (Web UI)](#usage-web-ui)  
6. [API Documentation](#api-documentation)  
7. [Examples](#examples)  
8. [Project Structure](#project-structure)  
9. [Troubleshooting](#troubleshooting)  
10. [Contributing](#contributing)  
11. [License](#license)  

---  

## Prerequisites
| Tool | Minimum Version | Why? |
|------|----------------|------|
| **Java JDK** | 11 (or newer) | Compile and run the servlet code |
| **Apache Tomcat** | 9.0+ | Servlet container |
| **MySQL Server** | 5.7+ | Database backend |
| **Maven** | 3.6+ (optional) | Build automation (if you prefer Maven) |
| **Git** | any | Clone the repository |
| **IDE** (IntelliJ IDEA, Eclipse, VS Code, …) | — | Helpful for debugging, not required |

> **Tip:** If you prefer Gradle over Maven, the project includes a `build.gradle` file that can be used interchangeably.

---  

## Installation  

### 1. Clone the repository
```bash
git clone https://github.com/yourusername/Mailbook.git
cd Mailbook
```

### 2. Set up the MySQL database  

```sql
-- Connect to MySQL as a user with CREATE DATABASE privileges
CREATE DATABASE mailbook CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

USE mailbook;

CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(150) NOT NULL UNIQUE,
    country VARCHAR(100) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### 3. Configure the DB connection  

Edit `src/main/resources/db.properties` (or `WEB-INF/classes/db.properties` after build) and set your credentials:

```properties
jdbc.driverClassName=com.mysql.cj.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/mailbook?useSSL=false&serverTimezone=UTC
jdbc.username=YOUR_DB_USER
jdbc.password=YOUR_DB_PASSWORD
```

> **Security note:** Do **not** commit real passwords. Use environment variables or a secret manager in production.

### 4. Build the project  

#### Using Maven
```bash
mvn clean package
```
The WAR file will be generated at `target/mailbook.war`.

#### Using Gradle
```bash
./gradlew clean war
```
The WAR file will be generated at `build/libs/mailbook.war`.

### 5. Deploy to Tomcat  

1. Copy the generated WAR to Tomcat’s `webapps` directory:
   ```bash
   cp target/mailbook.war $CATALINA_HOME/webapps/
   ```
2. Start (or restart) Tomcat:
   ```bash
   $CATALINA_HOME/bin/startup.sh   # Linux/macOS
   %CATALINA_HOME%\bin\startup.bat # Windows
   ```

Tomcat will automatically unpack the WAR and make the app available at `http://localhost:8080/mailbook`.

---  

## Configuration  

| Property | Description | Default |
|----------|-------------|---------|
| `jdbc.driverClassName` | MySQL JDBC driver class | `com.mysql.cj.jdbc.Driver` |
| `jdbc.url` | JDBC URL (include `useSSL` and `serverTimezone` params) | `jdbc:mysql://localhost:3306/mailbook?useSSL=false&serverTimezone=UTC` |
| `jdbc.username` | DB user | `root` |
| `jdbc.password` | DB password | (empty) |
| `app.contextPath` | Context path for the web app (usually `/mailbook`) | derived from WAR name |
| `log.level` | Log4j2 level (`INFO`, `DEBUG`, …) | `INFO` |

You can override any of these values with **system properties** or **environment variables**:

```bash
export DB_URL=jdbc:mysql://localhost:3306/mailbook?useSSL=false&serverTimezone=UTC
export DB_USER=myuser
export DB_PASS=mypassword
```

---  

## Running the Application  

| Step | Command / Action |
|------|------------------|
| **Start Tomcat** | `catalina.sh run` (Linux/macOS) or `catalina.bat run` (Windows) |
| **Open Browser** | `http://localhost:8080/mailbook/` |
| **Stop Tomcat** | `catalina.sh stop` (Linux/macOS) or `catalina.bat stop` (Windows) |

You should see the **Mailbook Home** page with a table of existing users (initially empty) and a form to add a new user.

---  

## Usage (Web UI)  

1. **Create a User** – Fill the *Name*, *Email*, and *Country* fields at the bottom of the page and click **Add**.  
2. **Edit a User** – Click the **Edit** button next to a row. The form will be pre‑filled; modify the fields and click **Update**.  
3. **Delete a User** – Click the **Delete** button next to a row. A JavaScript confirmation dialog appears; confirm to remove the record.  

All operations are performed via AJAX calls to the servlet layer, so the page never fully reloads.

---  

## API Documentation  

The API is exposed through a single servlet (`UserServlet`) that follows a **REST‑like** pattern. All endpoints return JSON and use standard HTTP status codes.

| HTTP Method | URL | Description | Request Parameters | Success Response (200) | Error Responses |
|-------------|-----|-------------|--------------------|------------------------|-----------------|
| **GET** | `/mailbook/api/users` | Retrieve all users | – | `[{ "id":1, "name":"John", "email":"john@example.com", "country":"USA", "created_at":"2024-09-01T12:34:56Z" }, …]` | `500 – Internal Server Error` |
| **GET** | `/mailbook/api/users/{id}` | Retrieve a single user | `id` (path) | `{ "id":1, "name":"John", … }` | `404 – Not Found`, `500` |
| **POST** | `/mailbook/api/users` | Create a new user | JSON body `{ "name":"...", "email":"...", "country":"..." }` | `201 – Created` + created object | `400 – Validation error`, `409 – Email already exists`, `500` |
| **PUT** | `/mailbook/api/users/{id}` | Update an existing user | `id` (path) + JSON body (any of `name`, `email`, `country`) | `200 – OK` + updated object | `400`, `404`, `409`, `500` |
| **DELETE** | `/mailbook/api/users/{id}` | Delete a user | `id` (path) | `204 – No Content` | `404`, `500` |

### Servlet Mapping (web.xml)

```xml
<servlet>
    <servlet-name>UserServlet</servlet-name>
    <servlet-class>com.mailbook.servlet.UserServlet</servlet-class>
</servlet>

<servlet-mapping>
    <servlet-name>UserServlet</servlet-name>
    <url-pattern>/api/users/*</url-pattern>
</servlet-mapping>
```

### Request / Response Examples  

#### Create a User (POST)

```http
POST /mailbook/api/users HTTP/1.1
Content-Type: application/json
Accept: application/json

{
  "name": "Alice Smith",
  "email": "alice.smith@example.com",
  "country": "Canada"
}
```

**Response**

```http
HTTP/1.1 201 Created
Content-Type: application/json

{
  "id": 5,
  "name": "Alice Smith",
  "email": "alice.smith@example.com",
  "country": "Canada",
  "created_at": "2025-09-24T08:15:30Z"
}
```

#### Update a User (PUT)

```http
PUT /mailbook/api/users/5 HTTP/1.1
Content-Type: application/json

{
  "country": "United Kingdom"
}
```

**Response**

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": 5,
 