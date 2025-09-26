# 📬 Mailbook  
**A JSP‑JDBC‑MySQL‑Servlet based Java project to demonstrate CRUD operations where users can create, read, update and delete their name, email and country.**  

---  

## Table of Contents
1. [Prerequisites](#prerequisites)  
2. [Installation](#installation)  
3. [Configuration](#configuration)  
4. [Running the Application](#running-the-application)  
5. [Usage (Web UI)](#usage-web-ui)  
6. [API Documentation](#api-documentation)  
7. [Examples (cURL & UI Walk‑through)](#examples)  
8. [Project Structure](#project-structure)  
9. [Troubleshooting & FAQ](#troubleshooting--faq)  
10. [Contributing](#contributing)  
11. [License](#license)  

---  

## Prerequisites
| Tool | Minimum Version | Why? |
|------|----------------|------|
| **Java JDK** | 11 (or higher) | Compile and run the servlet code |
| **Apache Tomcat** | 9.0+ | Servlet container |
| **MySQL Server** | 5.7+ | Database for persisting user data |
| **Maven** (optional) | 3.6+ | Build automation (if you prefer Maven) |
| **Git** | any | Clone the repository |
| **IDE** (IntelliJ IDEA, Eclipse, VS Code, …) | – | Helpful for debugging, but not required |

> **Tip:** If you prefer Gradle, the project also ships with a `build.gradle` file. The documentation below uses Maven commands, but you can replace them with the equivalent Gradle tasks (`./gradlew build`, `./gradlew war`).

---  

## Installation  

### 1. Clone the repository
```bash
git clone https://github.com/your‑org/Mailbook.git
cd Mailbook
```

### 2. Set up the MySQL database  

```sql
-- Connect to MySQL as a user with CREATE/DROP privileges
CREATE DATABASE IF NOT EXISTS mailbook CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

USE mailbook;

CREATE TABLE IF NOT EXISTS users (
    id        INT AUTO_INCREMENT PRIMARY KEY,
    name      VARCHAR(100) NOT NULL,
    email     VARCHAR(150) NOT NULL UNIQUE,
    country   VARCHAR(100) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

> **Optional:** Run the provided script `db/init.sql` (if present) to create the DB and table automatically:
```bash
mysql -u <user> -p < db/init.sql
```

### 3. Configure the DB connection  

Edit `src/main/resources/db.properties` (or `WEB-INF/classes/db.properties` after the build) and set your MySQL credentials:

```properties
jdbc.driverClassName=com.mysql.cj.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/mailbook?useSSL=false&serverTimezone=UTC
jdbc.username=your_mysql_user
jdbc.password=your_mysql_password
```

> **Security note:** Never commit real passwords. Add `db.properties` to `.gitignore` and keep a template file (`db.properties.example`) in the repo.

### 4. Build the WAR file  

```bash
# Using Maven
mvn clean package

# The generated WAR will be at target/mailbook.war
```

If you use Gradle:

```bash
./gradlew clean war
```

### 5. Deploy to Tomcat  

Copy the generated `mailbook.war` to Tomcat’s `webapps` directory:

```bash
cp target/mailbook.war $CATALINA_HOME/webapps/
```

Tomcat will automatically explode the WAR and start the application.  

> **Alternative (IDE):** Most IDEs let you add a Tomcat server and deploy the project directly from the IDE.

---  

## Configuration  

| Property | Description | Default |
|----------|-------------|---------|
| `jdbc.driverClassName` | Fully‑qualified driver class | `com.mysql.cj.jdbc.Driver` |
| `jdbc.url` | JDBC URL (include DB name, timezone, SSL options) | `jdbc:mysql://localhost:3306/mailbook?useSSL=false&serverTimezone=UTC` |
| `jdbc.username` | MySQL user | `root` |
| `jdbc.password` | MySQL password | (empty) |
| `app.contextPath` | Context path for the web app (usually `/mailbook`) | derived from WAR name |
| `log.level` | Log4j/SLF4J level (`INFO`, `DEBUG`, …) | `INFO` |

You can also override any of these values with environment variables (Tomcat supports `JNDI` resources) – see the `README-ENV.md` file for details.

---  

## Running the Application  

1. **Start Tomcat**  

   ```bash
   $CATALINA_HOME/bin/startup.sh
   ```

2. **Verify deployment**  

   Open a browser and navigate to:  

   ```
   http://localhost:8080/mailbook/
   ```

   You should see the **Mailbook Home** page with a table of existing users (empty on first run).

3. **Stop Tomcat**  

   ```bash
   $CATALINA_HOME/bin/shutdown.sh
   ```

---  

## Usage (Web UI)  

| Feature | UI Location | Description |
|---------|-------------|-------------|
| **List Users** | `/mailbook/` (home) | Shows a paginated table of all users. |
| **Create User** | `/mailbook/create.jsp` | Form with fields: *Name*, *Email*, *Country*. Submits a POST to `/mailbook/UserCreateServlet`. |
| **Edit User** | Click **Edit** on a row → `/mailbook/edit.jsp?id={id}` | Pre‑filled form; POSTs to `/mailbook/UserUpdateServlet`. |
| **Delete User** | Click **Delete** on a row → `/mailbook/UserDeleteServlet?id={id}` | Confirmation dialog (JS) → GET request deletes the record. |
| **Search** | Search box on home page | Filters the table by name/email (client‑side JS). |

> **Keyboard shortcuts** (optional):  
> - `n` → Open *Create* dialog  
> - `e` → Focus on *Edit* of the selected row  

---  

## API Documentation  

The application also exposes a **REST‑like** API (JSON) for programmatic access. All endpoints are under the context path `/mailbook/api`.

| HTTP Method | Endpoint | Request Body / Params | Success Response | Error Responses |
|-------------|----------|-----------------------|------------------|-----------------|
| **GET** | `/api/users` | `?page=1&size=20` (optional) | `200 OK` <br>```json { "users": [...], "total": 42, "page":1, "size":20 }``` | `500` – DB error |
| **GET** | `/api/users/{id}` | – | `200 OK` <br>```json { "id":1, "name":"John", "email":"john@example.com", "country":"USA" }``` | `404` – Not found |
| **POST** | `/api/users` | ```json { "name":"...", "email":"...", "country":"..." }``` | `201 Created` <br>```json { "id": 12, "message":"User created"} ``` | `400` – Validation, `409` – Duplicate email |
| **PUT** | `/api/users/{id}` | ```json { "name":"...", "email":"...", "country":"..." }``` | `200 OK` <br>```json { "message":"User updated"} ``` | `400`, `404` |
| **DELETE** | `/api/users/{id}` | – | `204 No Content` | `404` |

### Servlet Mapping (web.xml excerpt)

```xml
<servlet>
    <servlet-name>UserListServlet</servlet-name>
    <servlet-class>com.mailbook.servlet.UserListServlet</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>UserListServlet</servlet-name>
    <url-pattern>/api/users</url-pattern>
</servlet-mapping>

<!-- Similar mappings for Create, Update, Delete, etc. -->
```

### JSON Utility  

All API responses are generated by `JsonResponseUtil` which uses **Jackson** (`com.fasterxml.jackson.databind.ObjectMapper`).  

> **Note:** The UI still uses classic form posts (HTML) – the API is an extra convenience for integration tests or external clients.

---  

## Examples  

### 1. Using cURL (API)

```bash
# 1️⃣ List all users (first 10)
curl -s http://localhost:808