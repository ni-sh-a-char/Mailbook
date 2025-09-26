# Mailbook  
**A JSP‑JDBC‑MySQL‑Servlet based Java project to demonstrate CRUD operations where users can create, update, edit and delete their name, email and country respectively.**  

---  

## Table of Contents  

| # | Section |
|---|---------|
| 1 | [Project Overview](#1-project-overview) |
| 2 | [Prerequisites](#2-prerequisites) |
| 3 | [Installation](#3-installation) |
| 4 | [Configuration](#4-configuration) |
| 5 | [Running the Application](#5-running-the-application) |
| 6 | [Usage (Web UI)](#6-usage-web-ui) |
| 7 | [API Documentation](#7-api-documentation) |
| 8 | [Examples (cURL & JavaScript)](#8-examples) |
| 9 | [Project Structure](#9-project-structure) |
|10 | [Testing](#10-testing) |
|11 | [Troubleshooting](#11-troubleshooting) |
|12 | [Contributing](#12-contributing) |
|13 | [License](#13-license) |

---  

## 1. Project Overview  

Mailbook is a **pure Java EE** web application that showcases the classic **Create‑Read‑Update‑Delete (CRUD)** workflow using:

| Layer | Technology |
|-------|------------|
| **Presentation** | JSP, JSTL, Bootstrap 5 |
| **Business / Data Access** | Servlets + JDBC |
| **Persistence** | MySQL (via `mysql-connector-j`) |
| **Build** | Maven (or Gradle – see the *Alternative Build* section) |
| **Container** | Apache Tomcat 9+ (or any Servlet‑3.1+ container) |

The app stores a simple `users` table with the fields **id**, **name**, **email**, **country**. All operations are performed through a clean UI and are also exposed as REST‑like endpoints for programmatic access.

---  

## 2. Prerequisites  

| Tool | Minimum Version | Why |
|------|----------------|-----|
| **JDK** | 11 (or 17) | Language level, `var` support, modules |
| **Maven** | 3.6.3 | Build & dependency management (or Gradle 7.x) |
| **MySQL** | 5.7 / 8.0 | Database |
| **Tomcat** | 9.0 (Servlet 4.0) | Servlet container |
| **Git** | any | Clone the repo |
| **Browser** | Chrome/Firefox/Edge | UI testing |

> **Tip:** If you prefer Docker, see the *Docker Quick‑Start* section at the end of this file.

---  

## 3. Installation  

### 3.1 Clone the repository  

```bash
git clone https://github.com/your‑org/mailbook.git
cd mailbook
```

### 3.2 Build the project  

```bash
# Using Maven
mvn clean package

# The WAR file will be generated at:
#   target/mailbook-<version>.war
```

> **Alternative (Gradle)**  
> ```bash
> ./gradlew clean build
> # WAR at build/libs/mailbook-<version>.war
> ```

### 3.3 Set up the MySQL database  

```sql
-- Connect as root (or a privileged user)
CREATE DATABASE IF NOT EXISTS mailbook CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

USE mailbook;

CREATE TABLE IF NOT EXISTS users (
    id      INT AUTO_INCREMENT PRIMARY KEY,
    name    VARCHAR(100) NOT NULL,
    email   VARCHAR(150) NOT NULL UNIQUE,
    country VARCHAR(100) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### 3.4 Create a DB user (optional but recommended)  

```sql
CREATE USER 'mailbook_user'@'%' IDENTIFIED BY 'StrongPassword123!';
GRANT ALL PRIVILEGES ON mailbook.* TO 'mailbook_user'@'%';
FLUSH PRIVILEGES;
```

### 3.5 Deploy the WAR  

Copy the generated WAR to Tomcat’s `webapps` directory (or use the Tomcat manager).

```bash
cp target/mailbook-*.war $CATALINA_HOME/webapps/
```

Tomcat will automatically unpack the WAR and start the application.  

---  

## 4. Configuration  

All configurable values live in **`src/main/resources/db.properties`** (which is copied to `WEB-INF/classes` at build time).  

```properties
# -------------------------------------------------
# Database connection properties
# -------------------------------------------------
jdbc.driverClassName=com.mysql.cj.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/mailbook?useSSL=false&serverTimezone=UTC
jdbc.username=mailbook_user
jdbc.password=StrongPassword123!
```

### 4.1 Overriding at runtime  

You can override any property by defining a **system property** or an **environment variable** with the same name prefixed by `mailbook.`  

```bash
# Example: override DB URL
export mailbook.jdbc.url=jdbc:mysql://dbhost:3306/mailbook?useSSL=true
```

Tomcat will pick up the values automatically because `DBUtil` reads them via `System.getProperty` first, then falls back to the properties file.

---  

## 5. Running the Application  

| URL | Description |
|-----|-------------|
| `http://localhost:8080/mailbook/` | Home page – list of users |
| `http://localhost:8080/mailbook/create.jsp` | Form to add a new user |
| `http://localhost:8080/mailbook/edit.jsp?id=5` | Edit user with id=5 |
| `http://localhost:8080/mailbook/delete?id=5` | Delete user (GET for demo, POST in production) |
| `http://localhost:8080/mailbook/api/users` | JSON list of all users (GET) |
| `http://localhost:8080/mailbook/api/users` | Create a user (POST) |
| `http://localhost:8080/mailbook/api/users/{id}` | Get / Update / Delete a single user (GET, PUT, DELETE) |

Open a browser and navigate to the home page to start using the UI.

---  

## 6. Usage (Web UI)  

### 6.1 Create a user  

1. Click **“Add New User”** on the home page.  
2. Fill **Name**, **Email**, **Country**.  
3. Press **Save** – you’ll be redirected to the list view with a success toast.

### 6.2 Edit a user  

1. In the list, click the **pencil** icon next to the user you want to edit.  
2. Update the fields and click **Update**.  

### 6.3 Delete a user  

1. Click the **trash** icon.  
2. Confirm the JavaScript modal. The row disappears instantly (AJAX).  

### 6.4 Search & Pagination  

- The list page includes a **search box** (filters by name/email).  
- Pagination is handled server‑side (10 rows per page).  

---  

## 7. API Documentation  

All API endpoints return **JSON** and use standard HTTP status codes.  

| Method | Endpoint | Request Body | Success Response | Error Responses |
|--------|----------|--------------|------------------|-----------------|
| **GET** | `/mailbook/api/users` | – | `200 OK` → `[{id, name, email, country, created_at}, …]` | `500` – DB error |
| **GET** | `/mailbook/api/users/{id}` | – | `200 OK` → single user object | `404` – Not found |
| **POST** | `/mailbook/api/users` | `{ "name":"John", "email":"john@example.com", "country":"USA" }` | `201 Created` → created user with generated `id` | `400` – validation, `409` – duplicate email |
| **PUT** | `/mailbook/api/users/{id}` | Same as POST (any subset) | `200 OK` → updated user | `400`, `404` |
| **DELETE** | `/mailbook/api/users/{id}` | – | `204 No Content` | `404` |

### 7.1 Common Headers  

| Header | Value |
|--------|-------|
| `Content-Type` | `application/json` |
| `Accept` | `application/json` |
| `Cache-Control` | `no-cache` |

### 7.2 Servlet Mapping (web.xml)  

```xml
<servlet>
    <servlet-name>UserServlet</servlet-name>
    <servlet-class>com.mailbook.servlet.UserServlet</servlet-class>
</servlet>

<servlet-mapping>
