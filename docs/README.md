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
9. [Troubleshooting & FAQ](#troubleshooting--faq)  
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
| **Browser** | Modern (Chrome/Firefox/Edge) | Access the UI |

> **Tip:** If you use an IDE (IntelliJ IDEA, Eclipse, VS Code) you can let it manage the Maven/Gradle wrapper for you.

---  

## Installation  

### 1. Clone the repository
```bash
git clone https://github.com/your‑org/Mailbook.git
cd Mailbook
```

### 2. Set up the MySQL database  

```sql
-- Connect to MySQL as a user with CREATE privileges
CREATE DATABASE IF NOT EXISTS mailbook CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

USE mailbook;

CREATE TABLE IF NOT EXISTS users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(150) NOT NULL UNIQUE,
    country VARCHAR(100) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

> **Optional:** Run the provided script `db/init.sql` (if present) to create the DB and table automatically.

### 3. Configure DB connection  

Edit `src/main/resources/db.properties` (or `WEB-INF/classes/db.properties` after build) and set your MySQL credentials:

```properties
jdbc.driverClassName=com.mysql.cj.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/mailbook?useSSL=false&serverTimezone=UTC
jdbc.username=YOUR_DB_USER
jdbc.password=YOUR_DB_PASSWORD
```

### 4. Build the project  

#### Using Maven (recommended)
```bash
mvn clean package
```
The command produces a `mailbook.war` file under `target/`.

#### Using Gradle (if the repo contains a `build.gradle`)
```bash
./gradlew clean war
```

### 5. Deploy to Tomcat  

1. Copy the generated `mailbook.war` to `$CATALINA_HOME/webapps/`.  
2. Start (or restart) Tomcat:

```bash
$CATALINA_HOME/bin/startup.sh   # Linux/macOS
# or
$CATALINA_HOME\bin\startup.bat  # Windows
```

Tomcat will automatically explode the WAR and make the app available at `http://localhost:8080/mailbook`.

---  

## Configuration  

| Property | Description | Default |
|----------|-------------|---------|
| `jdbc.driverClassName` | Fully‑qualified driver class | `com.mysql.cj.jdbc.Driver` |
| `jdbc.url` | JDBC URL (include `useSSL` & `serverTimezone` params) | `jdbc:mysql://localhost:3306/mailbook?useSSL=false&serverTimezone=UTC` |
| `jdbc.username` | DB user | `root` |
| `jdbc.password` | DB password | (empty) |
| `app.pageSize` | Number of rows per page in the UI list view | `10` |

You can also override any of these values with environment variables (e.g., `DB_URL`, `DB_USER`, `DB_PASSWORD`) – the `DBUtil` class checks the environment first.

---  

## Running the Application  

| Step | Action |
|------|--------|
| **Start Tomcat** | `startup.sh` / `startup.bat` |
| **Open Browser** | `http://localhost:8080/mailbook/` |
| **Login** | No authentication – the app is public for demo purposes |
| **Navigate** | Use the navigation bar to go to **Add User**, **User List**, etc. |

---  

## Usage (Web UI)  

### 1. Add a New User  
- **URL:** `http://localhost:8080/mailbook/add.jsp`  
- Fill **Name**, **Email**, **Country** → **Submit**.  
- Validation: non‑empty fields, email format, unique email.

### 2. List Users  
- **URL:** `http://localhost:8080/mailbook/list.jsp`  
- Pagination controls at the bottom (page size configurable).  
- Each row shows **Edit** and **Delete** icons.

### 3. Edit a User  
- Click the **Edit** icon → `edit.jsp?id=XX`.  
- Form pre‑populated; modify fields → **Update**.

### 4. Delete a User  
- Click the **Delete** icon → confirmation dialog → **OK**.  
- The servlet performs a `DELETE FROM users WHERE id=?`.

---  

## API Documentation  

The project also exposes a simple REST‑like API (implemented as servlets). All endpoints return **JSON** and use the standard HTTP status codes.

| HTTP Method | Endpoint | Description | Request Parameters | Success Response | Error Responses |
|-------------|----------|-------------|--------------------|------------------|-----------------|
| `GET` | `/mailbook/api/users` | Retrieve a paginated list of users | `page` (optional, default 1) `size` (optional, default 10) | `200 OK` <br>`{ "page":1, "size":10, "total":42, "users":[{...}] }` | `500 Internal Server Error` |
| `GET` | `/mailbook/api/users/{id}` | Get a single user by ID | – | `200 OK` <br>`{ "id":5, "name":"John", "email":"john@example.com", "country":"USA" }` | `404 Not Found` |
| `POST` | `/mailbook/api/users` | Create a new user | JSON body `{ "name":"...", "email":"...", "country":"..." }` | `201 Created` <br>`{ "id": 43, "message":"User created" }` | `400 Bad Request` (validation) <br>`409 Conflict` (duplicate email) |
| `PUT` | `/mailbook/api/users/{id}` | Update an existing user | JSON body (any of the three fields) | `200 OK` <br>`{ "message":"User updated" }` | `400 Bad Request` <br>`404 Not Found` |
| `DELETE` | `/mailbook/api/users/{id}` | Delete a user | – | `204 No Content` | `404 Not Found` |

### Servlet Mapping (web.xml excerpt)

```xml
<servlet>
    <servlet-name>UserServlet</servlet-name>
    <servlet-class>com.mailbook.controller.UserServlet</servlet-class>
</servlet>

<servlet-mapping>
    <servlet-name>UserServlet</servlet-name>
    <url-pattern>/api/users/*</url-pattern>
</servlet-mapping>
```

### JSON Utility  

All responses are generated via `JsonUtil` (uses `com.google.gson.Gson`). Errors are wrapped in:

```json
{
  "timestamp": "2025-09-27T14:32:10Z",
  "status": 404,
  "error": "Not Found",
  "message": "User with id 99 does not exist",
  "path": "/mailbook/api/users/99"
}
```

---  

## Examples  

### 1. Create a user (cURL)

```bash
curl -X POST http://localhost:8080/mailbook/api/users \
     -H "Content-Type: application/json" \
     -d '{"name":"Alice Smith","email":"alice@example.com","country":"Canada"}'
```

**Response**

```json
{
  "id": 57,
  "message": "User created"
}
```

### 2. Get a paginated list (cURL)

```bash
curl http://localhost:8080/mailbook/api/users?page=2&size=5
