# Mailbook  
**A JSP‑JDBC‑MySQL‑Servlet based Java project to demonstrate CRUD operations where users can create, update, edit and delete their name, email and country respectively.**  

---  

## Table of Contents
1. [Prerequisites](#prerequisites)  
2. [Installation](#installation)  
   - 2.1 [Clone the repository](#clone-the-repository)  
   - 2.2 [Set up the MySQL database](#set-up-the-mysql-database)  
   - 2.3 [Configure the project](#configure-the-project)  
   - 2.4 [Build & Deploy](#build--deploy)  
3. [Usage](#usage)  
   - 3.1 [Running the application](#running-the-application)  
   - 3.2 [Web UI walkthrough](#web-ui-walkthrough)  
4. [API Documentation](#api-documentation)  
   - 4.1 [Endpoints Overview](#endpoints-overview)  
   - 4.2 [Request / Response Formats](#request--response-formats)  
5. [Examples](#examples)  
   - 5.1 [Create a user (UI)](#create-a-user-ui)  
   - 5.2 [Create a user (cURL)](#create-a-user-curl)  
   - 5.3 [Read / List users (UI & cURL)](#read--list-users)  
   - 5.4 [Update a user (UI & cURL)](#update-a-user)  
   - 5.5 [Delete a user (UI & cURL)](#delete-a-user)  
6. [Project Structure](#project-structure)  
7. [Troubleshooting & FAQ](#troubleshooting--faq)  
8. [Contributing](#contributing)  
9. [License](#license)  

---  

## Prerequisites
| Tool | Minimum version | Why it’s needed |
|------|----------------|-----------------|
| **Java JDK** | 11 (or newer) | Compile and run the servlet code |
| **Apache Tomcat** | 9.0+ | Servlet container for deployment |
| **MySQL Server** | 5.7+ | Persistent storage for user data |
| **Maven** (optional) | 3.6+ | Build automation (alternatively use the provided Ant script) |
| **Git** | any | Clone the repository |
| **Browser** | modern (Chrome/Firefox/Edge) | Interact with the UI |

> **Tip:** If you prefer an IDE, IntelliJ IDEA Community/Eclipse works out‑of‑the‑box with Maven support.

---  

## Installation  

### 1. Clone the repository
```bash
git clone https://github.com/your‑org/Mailbook.git
cd Mailbook
```

### 2. Set up the MySQL database  

1. **Start MySQL** and log in as a user with `CREATE DATABASE` privileges.
   ```bash
   mysql -u root -p
   ```

2. **Create the database and table** (the `schema.sql` file is shipped in the repo):
   ```sql
   SOURCE src/main/resources/schema.sql;
   ```

   The script creates a database called `mailbook_db` and a table `users`:

   ```sql
   CREATE DATABASE IF NOT EXISTS mailbook_db;
   USE mailbook_db;

   CREATE TABLE IF NOT EXISTS users (
       id INT AUTO_INCREMENT PRIMARY KEY,
       name VARCHAR(100) NOT NULL,
       email VARCHAR(150) NOT NULL UNIQUE,
       country VARCHAR(100) NOT NULL,
       created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
   );
   ```

3. **Create a MySQL user** (optional but recommended) and grant privileges:
   ```sql
   CREATE USER 'mailbook_user'@'localhost' IDENTIFIED BY 'StrongPassword123!';
   GRANT ALL PRIVILEGES ON mailbook_db.* TO 'mailbook_user'@'localhost';
   FLUSH PRIVILEGES;
   ```

### 3. Configure the project  

Edit the **`src/main/resources/db.properties`** file (or `WEB-INF/classes/db.properties` after build) to match your DB credentials:

```properties
jdbc.driverClassName=com.mysql.cj.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/mailbook_db?useSSL=false&serverTimezone=UTC
jdbc.username=mailbook_user
jdbc.password=StrongPassword123!
```

> **Note:** If you are using a different host/port, adjust the `jdbc.url` accordingly.

### 4. Build & Deploy  

#### Using Maven (recommended)

```bash
# Clean and package the WAR
mvn clean package

# The generated WAR will be at target/mailbook.war
# Deploy it to Tomcat
cp target/mailbook.war $CATALINA_HOME/webapps/
```

Tomcat will automatically unpack the WAR and start the application.  

#### Using Ant (if you prefer)

```bash
ant clean war
cp build/mailbook.war $CATALINA_HOME/webapps/
```

> **After deployment**, open Tomcat’s manager (`http://localhost:8080/manager/html`) to verify that `mailbook` is listed as **Running**.

---  

## Usage  

### Running the application  

1. **Start Tomcat** (if not already running):
   ```bash
   $CATALINA_HOME/bin/startup.sh   # Linux/macOS
   %CATALINA_HOME%\bin\startup.bat # Windows
   ```

2. **Open the web UI** in a browser:
   ```
   http://localhost:8080/mailbook/
   ```

You should see the **Mailbook Home** page with a table of existing users (empty on first run) and a **“Add New User”** button.

### Web UI walkthrough  

| UI Element | Description |
|------------|-------------|
| **Add New User** button | Opens a modal (or separate JSP) where you can input *Name*, *Email*, and *Country*. Submits a `POST` to `/user/create`. |
| **Edit** link (per row) | Loads the same form pre‑filled with the selected user’s data. Submits a `POST` to `/user/update`. |
| **Delete** link (per row) | Sends a `GET` request to `/user/delete?id={id}` after a JavaScript confirmation dialog. |
| **Search** box (optional) | Filters the table client‑side (pure JavaScript). No server call. |
| **Pagination** (if many rows) | Handled by the servlet using `LIMIT`/`OFFSET`. |

---  

## API Documentation  

All endpoints are served under the context path **`/mailbook`** (the WAR name). The servlet mapping is defined in `WEB-INF/web.xml`.

### 4.1 Endpoints Overview  

| HTTP Method | URL Pattern | Description | Parameters | Returns |
|-------------|-------------|-------------|------------|---------|
| `GET` | `/mailbook/users` | List all users (HTML view) | – | `users.jsp` |
| `GET` | `/mailbook/user/create` | Show the “Create User” form | – | `user-form.jsp` |
| `POST` | `/mailbook/user/create` | Persist a new user | `name`, `email`, `country` (form‑encoded) | Redirect → `/mailbook/users` |
| `GET` | `/mailbook/user/edit` | Show the “Edit User” form | `id` (query) | `user-form.jsp` (pre‑filled) |
| `POST` | `/mailbook/user/update` | Update an existing user | `id`, `name`, `email`, `country` | Redirect → `/mailbook/users` |
| `GET` | `/mailbook/user/delete` | Delete a user | `id` (query) | Redirect → `/mailbook/users` |
| `GET` | `/mailbook/api/users` | **REST** – JSON list of all users | – | `application/json` |
| `GET` | `/mailbook/api/user/{id}` | **REST** – JSON of a single user | Path variable `id` | `application/json` |
| `POST` | `/mailbook/api/user` | **REST** – Create user (JSON body) | `{ "name": "...", "email": "...", "country": "..." }` | `201 Created`, JSON of created user |
| `PUT` | `/mailbook/api/user/{id}` | **REST** – Update user (JSON body) | Path variable `id`; JSON payload with fields to update | `200 OK`, updated user JSON |
| `DELETE` | `/mailbook/api/user/{id}` | **REST** – Delete user | Path variable `id` | `204 No Content` |

> **Note:** The UI uses the servlet‑based HTML endpoints, while the `/api/*` endpoints are provided for programmatic access (e.g