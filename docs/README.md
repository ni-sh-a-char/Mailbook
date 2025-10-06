# 📬 Mailbook  
**A JSP‑JDBC‑MySQL‑Servlet based Java project to demonstrate CRUD operations where users can create, read, update and delete their name, email and country.**  

---  

## Table of Contents
1. [Prerequisites](#prerequisites)  
2. [Installation](#installation)  
   - [1️⃣ Clone the repo](#clone)  
   - [2️⃣ Set up MySQL](#mysql-setup)  
   - [3️⃣ Configure the project](#configuration)  
   - [4️⃣ Build & Deploy](#build-deploy)  
3. [Usage](#usage)  
   - [Running the application](#run)  
   - [Web UI walkthrough](#ui-walkthrough)  
4. [API Documentation](#api-documentation)  
   - [Servlet End‑points](#servlet-endpoints)  
   - [Request / Response formats](#request-response)  
5. [Examples](#examples)  
   - [Using the UI](#ui-examples)  
   - [cURL / HTTP client examples](#curl-examples)  
6. [Project Structure](#project-structure)  
7. [Troubleshooting & FAQ](#troubleshooting)  
8. [Contributing](#contributing)  
9. [License](#license)  

---  

<a name="prerequisites"></a>
## 1️⃣ Prerequisites
| Tool | Minimum version | Why? |
|------|----------------|------|
| **Java JDK** | 11 (or 17) | Compile & run the servlet code |
| **Apache Tomcat** | 9.0+ | Servlet container |
| **MySQL Server** | 5.7+ | Database for persisting users |
| **Maven** | 3.6+ | Build automation (optional – you can use the provided Ant script) |
| **Git** | any | Clone the repository |
| **Browser** | Modern (Chrome/Firefox/Edge) | UI testing |

> **Tip:** If you prefer Docker, a `docker-compose.yml` is provided in the `docker/` folder (see [Docker Quick‑Start](#docker-quick-start) below).

---  

<a name="installation"></a>
## 2️⃣ Installation  

<a name="clone"></a>
### 2.1 Clone the repository
```bash
git clone https://github.com/your‑org/Mailbook.git
cd Mailbook
```

<a name="mysql-setup"></a>
### 2.2 Set up MySQL  

1. **Create a database** (you can name it anything, e.g., `mailbook_db`):
   ```sql
   CREATE DATABASE mailbook_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
   ```

2. **Create the `users` table** – the project ships a ready‑to‑run script:
   ```bash
   mysql -u root -p mailbook_db < sql/create_tables.sql
   ```

   `create_tables.sql` contains:
   ```sql
   CREATE TABLE users (
       id INT AUTO_INCREMENT PRIMARY KEY,
       name VARCHAR(100) NOT NULL,
       email VARCHAR(150) NOT NULL UNIQUE,
       country VARCHAR(100) NOT NULL,
       created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
   );
   ```

3. **Create a MySQL user** (optional but recommended):
   ```sql
   CREATE USER 'mailbook_user'@'%' IDENTIFIED BY 'StrongPassword123!';
   GRANT ALL PRIVILEGES ON mailbook_db.* TO 'mailbook_user'@'%';
   FLUSH PRIVILEGES;
   ```

<a name="configuration"></a>
### 2.3 Configure the project  

Edit `src/main/resources/db.properties` (or `WEB-INF/classes/db.properties` after build) with your DB credentials:

```properties
jdbc.driverClassName=com.mysql.cj.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/mailbook_db?useSSL=false&serverTimezone=UTC
jdbc.username=mailbook_user
jdbc.password=StrongPassword123!
```

> **If you are using Tomcat’s JNDI datasource**, set the JNDI name in `web.xml` and comment out the `db.properties` block.

<a name="build-deploy"></a>
### 2.4 Build & Deploy  

#### Option A – Maven (recommended)

```bash
# Clean & package the WAR
mvn clean package

# The WAR will be at target/mailbook.war
# Deploy to Tomcat (copy or use Tomcat manager)
cp target/mailbook.war $CATALINA_HOME/webapps/
```

#### Option B – Ant (legacy)

```bash
ant clean war
cp build/mailbook.war $CATALINA_HOME/webapps/
```

#### Option C – Docker (quick start)

```bash
cd docker
docker-compose up -d
# The app will be reachable at http://localhost:8080/mailbook
```

> **Docker compose** spins up three containers: `tomcat`, `mysql`, and `adminer` (optional DB UI).  
> The `docker/.env` file contains default credentials – edit it before first run.

---  

<a name="usage"></a>
## 3️⃣ Usage  

<a name="run"></a>
### 3.1 Running the application  

After deployment, start Tomcat (if not already running):

```bash
$CATALINA_HOME/bin/startup.sh
```

Open a browser and navigate to:

```
http://localhost:8080/mailbook/
```

You should see the **Mailbook Home** page with a table of existing users (initially empty) and a **“Add New User”** button.

<a name="ui-walkthrough"></a>
### 3.2 Web UI Walkthrough  

| Action | UI Element | What Happens |
|--------|------------|--------------|
| **Create** | *Add New User* button → modal form | Submits a POST to `/user/create` → inserts a row → UI refreshes |
| **Read** | Users table rows | Data fetched via `UserListServlet` (GET) and displayed |
| **Update** | *Edit* icon on a row → modal pre‑filled | Submits a POST to `/user/update` → updates DB → UI refreshes |
| **Delete** | *Trash* icon on a row → confirm dialog | Sends a GET to `/user/delete?id=XX` → deletes row → UI refreshes |

All UI interactions are powered by **jQuery AJAX** (see `webapp/js/main.js`). No full‑page reload is required.

---  

<a name="api-documentation"></a>
## 4️⃣ API Documentation  

The core of Mailbook is a set of **Servlets** that expose CRUD functionality via HTTP. They are deliberately simple to illustrate JDBC usage; they are **not** a full‑blown REST API, but they can be consumed by any HTTP client.

<a name="servlet-endpoints"></a>
### 4.1 Servlet End‑points  

| HTTP Method | URL | Description | Parameters |
|-------------|-----|-------------|------------|
| `GET` | `/mailbook/` | Home page (JSP) | – |
| `GET` | `/mailbook/users` | Returns JSON list of all users | – |
| `POST` | `/mailbook/user/create` | Insert a new user | `name`, `email`, `country` |
| `POST` | `/mailbook/user/update` | Update an existing user | `id`, `name`, `email`, `country` |
| `GET` | `/mailbook/user/delete` | Delete a user by id | `id` (query string) |
| `GET` | `/mailbook/user/edit` | Fetch a single user (JSON) for edit modal | `id` |

> **All POST requests must include the `Content-Type: application/x-www-form-urlencoded` header** (the default for HTML forms).  

> **All responses are JSON** (except the JSP pages). Errors are returned with HTTP status `400` or `500` and a JSON body:  

```json
{
  "status": "error",
  "message": "Detailed error description"
}
```

<a name="request-response"></a>
### 4.2 Request / Response Formats  

#### 4.2.1 List Users (`GET /mailbook/users`)

**Response (200 OK)**
```json
{
  "status": "success",
  "data": [
    {
      "id": 1,
      "name": "Alice",
      "email": "alice@example.com",
      "country": "USA",
      "created_at": "2024-09-15T12:34:56Z"
    },
    {
      "id": 2,
      "name": "Bob",
      "email": "bob@example.org",
      "country": "Canada",
      "created_at": "2024-09-16T08:22:10Z"
    }
  ]
}
```

#### 4.2.2 Create User (`POST /mailbook/user/create`)

**Request body (form‑urlencoded)**
