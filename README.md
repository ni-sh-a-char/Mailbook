# JSP-JDBC-MySQL-Servlet CRUD Operations Demo

## Setup and Deployment

To set up and deploy the project, follow these steps:

1. Update the database connection details in the `db.properties` file located in the `src/main/resources` directory. Modify the values for `url`, `username`, and `password` according to your MySQL setup.

2. Build the project using your IDE or by running the Maven command.

3. Deploy the project to the Apache Tomcat server.

4. Access the application by navigating to [http://localhost:8080/crud-demo](http://localhost:8080/crud-demo) in your web browser.

## Functionality

The application provides the following functionality:

- Create a new user: Users can enter their name, email, and country information to create a new user record.
- Read user information: The application displays a list of all users and their respective details.
- Update user information: Users can edit their existing information, including name, email, and country.
- Delete a user: Users can delete their record from the database.

## Technologies Used

The project utilizes the following technologies:

- JSP (JavaServer Pages): Used for rendering dynamic web content.
- JDBC (Java Database Connectivity): Used for database connectivity and executing SQL queries.
- MySQL: The relational database management system.
- Servlet: Handles HTTP requests and responses.

## Project Structure

The project structure follows the standard Maven web application structure. The key directories and files are as follows:

- `src/main/java`: Contains the Java source files, including servlets and database utility classes.
- `src/main/webapp`: Contains JSP files, CSS files, and the web deployment descriptor (`web.xml`).

## Conclusion

This Java project provides a demonstration of CRUD operations using JSP, JDBC, MySQL, and Servlet. You can use this as a reference or starting point to build similar applications or integrate the provided functionality into your existing projects.
