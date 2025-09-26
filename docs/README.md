# Mailbook  
**A JSP‑JDBC‑MySQL‑Servlet based Java project to demonstrate CRUD operations where users can create, read, update and delete their name, email and country.**  

---  

## Table of Contents
1. [Prerequisites](#prerequisites)  
2. [Installation](#installation)  
   - 2.1 [Clone the repository](#clone-the-repository)  
   - 2.2 [Database setup](#database-setup)  
   - 2.3 [Configure the application](#configure-the-application)  
   - 2.4 [Build & Deploy](#build--deploy)  
3. [Usage](#usage)  
   - 3.1 [Running the app](#running-the-app)  
   - 3.2 [Web UI walkthrough](#web-ui-walkthrough)  
4. [API Documentation](#api-documentation)  
   - 4.1 [Endpoints Overview](#endpoints-overview)  
   - 4.2 [Request / Response Formats](#request--response-formats)  
5. [Examples](#examples)  
   - 5.1 [cURL examples](#curl-examples)  
   - 5.2 [JSP snippet for a custom form](#jsp-snippet-for-a-custom-form)  
6. [Testing](#testing)  
7. [Contributing](#contributing)  
8. [License](#license)  

---  

## Prerequisites
| Tool | Minimum Version | Why |
|------|----------------|-----|
| **Java JDK** | 11 (or newer) | Compile and run the servlet code |
| **Apache Tomcat** | 9.0+ | Servlet container |
| **MySQL Server** | 5.7+ | Persistence layer |
| **Maven** | 3.6+ | Build automation (optional – you can use the provided Ant script) |
| **Git** | any | Clone the repo |
| **Browser** | Chrome/Firefox/Edge | UI interaction |

> **Tip:** If you prefer Docker, a `docker-compose.yml` is provided in the `docker/` folder (see [Docker Setup](#docker-setup) below).

---  

## Installation  

### 1. Clone the repository
```bash
git clone https://github.com/yourusername/Mailbook.git
cd Mailbook
```

### 2. Database setup  

1. **Start MySQL** and create a database called `mailbook`:

   ```sql
   CREATE DATABASE mailbook CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
   ```

2. **Create the `users` table** (the schema is in `src/main/resources/sql/schema.sql`):

   ```sql
   USE mailbook;
   SOURCE src/main/resources/sql/schema.sql;
   ```

   The table definition:

   ```sql
   CREATE TABLE users (
       id INT AUTO_INCREMENT PRIMARY KEY,
       name VARCHAR(100) NOT NULL,
       email VARCHAR(150) NOT NULL UNIQUE,
       country VARCHAR(100) NOT NULL,
       created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
   );
   ```

3. (Optional) **Insert sample data**:

   ```sql
   INSERT INTO users (name, email, country) VALUES
   ('Alice Johnson', 'alice@example.com', 'USA'),
   ('Bob Smith', 'bob@example.com', 'Canada');
   ```

### 3. Configure the application  

Edit `src/main/resources/config.properties` (or `WEB-INF/classes/config.properties` after build) to match your MySQL credentials:

```properties
# Database connection
db.url=jdbc:mysql://localhost:3306/mailbook?useSSL=false&serverTimezone=UTC
db.username=your_mysql_user
db.password=your_mysql_password
```

> **Security note:** Do **not** commit real passwords. Use environment variables or a secret manager in production.

### 4. Build & Deploy  

#### Using Maven (recommended)

```bash
mvn clean package
```

The command produces a `target/mailbook.war`. Deploy it to Tomcat:

```bash
# Assuming TOMCAT_HOME points to your Tomcat installation
cp target/mailbook.war $TOMCAT_HOME/webapps/
```

Tomcat will automatically explode the WAR and start the application.  

#### Using Ant (legacy)

```bash
ant clean war
cp build/mailbook.war $TOMCAT_HOME/webapps/
```

#### Docker (quick start)

```bash
docker-compose -f docker/docker-compose.yml up -d
```

The compose file spins up a MySQL container, a Tomcat container (with the WAR mounted), and sets the required environment variables.

---  

## Usage  

### Running the app  

Open a browser and navigate to:

```
http://localhost:8080/mailbook/
```

You should see the **Mailbook Home** page with a table listing all users and buttons for **Create**, **Edit**, **Delete**.

### Web UI walkthrough  

| Action | UI Element | What Happens |
|--------|------------|--------------|
| **Create a user** | *Create New* button → opens `create.jsp` | Fill **Name**, **Email**, **Country** → Submit → Servlet `UserCreateServlet` inserts the row and redirects back to the list. |
| **Read / List** | Main table on `index.jsp` | Shows all rows ordered by `created_at DESC`. |
| **Update a user** | *Edit* icon next to a row → opens `edit.jsp?id=XX` | Form pre‑filled → Submit → `UserUpdateServlet` updates the row. |
| **Delete a user** | *Delete* icon next to a row → confirmation dialog → `UserDeleteServlet` removes the row. |
| **Search** | Search box on top of the table | Sends a GET request to `UserSearchServlet?query=...` and refreshes the table with matching rows. |

---  

## API Documentation  

All endpoints are **Servlet‑based** and return **HTML** for the UI. However, each servlet also supports **JSON** when the request header `Accept: application/json` is present. This makes it easy to integrate with external clients.

| HTTP Method | URL | Servlet | Description | Parameters | Success Response |
|-------------|-----|---------|-------------|------------|------------------|
| `GET` | `/mailbook/` | `UserListServlet` | List all users | – | HTML page (`index.jsp`) or JSON array |
| `GET` | `/mailbook/create.jsp` | – | Render create‑form | – | HTML form |
| `POST` | `/mailbook/create` | `UserCreateServlet` | Insert a new user | `name`, `email`, `country` (form‑encoded) | Redirect to `/` (HTML) or JSON `{status:"ok", id:XX}` |
| `GET` | `/mailbook/edit.jsp` | – | Render edit‑form | `id` (query) | HTML form pre‑filled |
| `POST` | `/mailbook/update` | `UserUpdateServlet` | Update an existing user | `id`, `name`, `email`, `country` | Redirect to `/` or JSON `{status:"ok"}` |
| `POST` | `/mailbook/delete` | `UserDeleteServlet` | Delete a user | `id` | Redirect to `/` or JSON `{status:"ok"}` |
| `GET` | `/mailbook/search` | `UserSearchServlet` | Search by name/email/country | `query` | HTML table or JSON array |
| `GET` | `/mailbook/api/users` | `UserApiServlet` | REST‑style endpoint (JSON only) | `page`, `size` (optional) | `{ "page":1, "size":10, "total":42, "data":[...] }` |

### Request / Response Formats  

#### HTML (default)  

- **Content‑Type:** `text/html;charset=UTF-8`  
- Rendered via JSP pages located under `WEB-INF/jsp/`.  

#### JSON (when `Accept: application/json`)  

- **Content‑Type:** `application/json;charset=UTF-8`  
- Standard response envelope:

```json
{
  "status": "ok",          // "ok" | "error"
  "message": "Optional human‑readable text",
  "data": {...}            // payload (object, array, or null)
}
```

- Errors return HTTP status `400` or `500` with:

```json
{
  "status": "error",
  "message": "Detailed error description"
}
```

---  

## Examples  

### 1. cURL examples  

#### Create a new user (JSON)

```bash
curl -X POST http://localhost:8080/mailbook/create \
     -H "Content-Type: application/x-www-form-urlencoded" \
     -H "Accept: application/json" \
     -d "name=John+Doe&email=john.doe%40example