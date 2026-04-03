# SplitSmart

SplitSmart is a beginner-friendly Spring Boot REST API for splitting shared expenses in a group, similar to a basic Splitwise clone.

The application lets you:

- register users
- create groups
- add users to groups
- record expenses for a group
- calculate who should pay whom to settle balances

## Tech Stack

- Java 17
- Spring Boot 3.2.5
- Spring Web
- Spring Data JPA
- Spring Security
- MySQL
- Maven

## How The Business Logic Works

The app is built around 4 main entities:

- `User`: a person using the app
- `Group`: a set of users who share expenses
- `Expense`: a bill paid by one user for one group
- `Split`: the per-user breakdown of an expense

When an expense is added:

1. The app loads the payer and the group.
2. It saves the expense.
3. It gets all group members.
4. It divides the total amount equally across all members.
5. It creates one `Split` row per member.

Example:

- Alice pays `300`
- Group has 3 members: Alice, Bob, Carol
- Each member owes `100`

Stored result:

- Alice paid `300`, owes `100`, net `+200`
- Bob paid `0`, owes `100`, net `-100`
- Carol paid `0`, owes `100`, net `-100`

The settlement API sums those balances and returns:

- Bob pays Alice `100`
- Carol pays Alice `100`

## Project Structure

Use the root project folder:

- `C:\Users\LENOVO\Downloads\splitsmart`

Important folders:

- `src/main/java/com/splitsmart`
- `src/main/resources`
- `target`

Main code layout:

- `controller`: REST endpoints
- `service`: business logic
- `repository`: database access
- `entity`: database models
- `dto`: request/response objects
- `security`: Spring Security config

Important files:

- `src/main/java/com/splitsmart/SplitSmartApplication.java`: app entry point
- `src/main/resources/application.properties`: database settings
- `src/main/java/com/splitsmart/service/ExpenseService.java`: core expense and settlement logic

Note:

- There is also a nested duplicate folder named `splitsmart` inside the root project.
- For normal use, open and run the root folder, not the nested copy.

## Prerequisites

Install these before running the project:

- JDK 17 or newer
- Maven 3.9 or newer
- MySQL 8 or newer

Notes:

- The `pom.xml` targets Java 17.
- This repository does not include `mvnw` or `mvnw.cmd`, so Maven must be installed separately.

## Database Setup

The app expects a local MySQL database named `splitsmart`.

Current config in `src/main/resources/application.properties`:

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/splitsmart
spring.datasource.username=root
spring.datasource.password=

spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
```

Create the database in MySQL:

```sql
CREATE DATABASE splitsmart;
```

If your MySQL username or password is different, update `application.properties`.

Because `spring.jpa.hibernate.ddl-auto=update` is enabled, Hibernate will create or update the tables automatically when the app starts.

## How To Run

Open a terminal in the project root:

```powershell
cd C:\Users\LENOVO\Downloads\splitsmart
```

Run the app:

```powershell
mvn spring-boot:run
```

The application will start on:

- `http://localhost:8080`

You can also run it from an IDE:

- IntelliJ IDEA: open the root folder as a Maven project, then run `SplitSmartApplication`
- Eclipse: import it as an Existing Maven Project, then run `SplitSmartApplication`

## Security And Authentication

This project includes Spring Security with HTTP Basic authentication.

Current behavior:

- `POST /users/register` is public
- all other endpoints require authentication

Important beginner note:

- Registering a user in the database does not log you in
- The current code does not connect registered users to Spring Security authentication
- If no custom user is configured, Spring Boot usually creates a default user named `user` and prints a generated password in the startup logs

For testing protected endpoints, check the console output after startup for a line similar to:

```text
Using generated security password: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

Then use:

- username: `user`
- password: the generated password from the console

## Recommended Beginner Test Flow

Test the app in this order:

1. Register users
2. Create a group
3. Add users to the group
4. Add an expense
5. Get settlement output

## API Endpoints

### Users

`POST /users/register`

- Public endpoint
- Creates a new user

Example request body:

```json
{
  "name": "Het",
  "email": "het@gmail.com",
  "password": "123",
  "role": "ROLE_USER"
}
```

```json
{
  "name": "Riddhi",
  "email": "riddhi@gmail.com",
  "password": "123",
  "role": "ROLE_USER"
}
```

```json
{
  "name": "Rutvi",
  "email": "rutvi@gmail.com",
  "password": "123",
  "role": "ROLE_USER"
}
```

`GET /users`

- Protected endpoint
- Returns all users

### Groups

`POST /groups`

- Protected endpoint
- Creates a group

Example request body:

```json
{
  "name": "Manali Trip"
}
```

`POST /groups/{groupId}/users`

- Protected endpoint
- Sets group members using a list of user IDs

Example request body:

```json
[1, 2, 3]
```

`GET /groups`

- Protected endpoint
- Returns all groups

### Expenses

`POST /expenses`

- Protected endpoint
- Creates an expense and automatically creates split rows

Example request body:

```json
{
  "description": "Bungee Jumping",
  "amount": 2700,
  "paidByUserId": 1,
  "groupId": 1
}
```

```json
{
  "description": "Maggie",
  "amount": 200,
  "paidByUserId": 2,
  "groupId": 1
}
```

`GET /expenses/group/{groupId}`

- Protected endpoint
- Returns all expenses for a group

`GET /expenses/settlements/{groupId}`

- Protected endpoint
- Returns final settlement instructions for the group

`PUT /expenses/{expenseId}`

- Protected endpoint
- Updates an expense and rebuilds its split rows

```json
{

  "description": "Bungee Jumping Updated",
  "amount": 3000,
  "paidByUserId": 2,
  "groupId": 1
}
```

`DELETE /expenses/{expenseId}`

- Protected endpoint
- Deletes the expense and its related split rows

## Example Requests

These examples are written in a simple REST client style.

Important:

- `POST /users/register` accepts one user object per request
- `POST /expenses` accepts one expense object per request
- for protected endpoints, add Basic Auth in your API client

### USER ADD

Run these as 3 separate requests:

```http
POST http://localhost:8080/users/register
Content-Type: application/json

{
  "name": "Het",
  "email": "het@gmail.com",
  "password": "123",
  "role": "ROLE_USER"
}
```

```http
POST http://localhost:8080/users/register
Content-Type: application/json

{
  "name": "Riddhi",
  "email": "riddhi@gmail.com",
  "password": "123",
  "role": "ROLE_USER"
}
```

```http
POST http://localhost:8080/users/register
Content-Type: application/json

{
  "name": "Rutvi",
  "email": "rutvi@gmail.com",
  "password": "123",
  "role": "ROLE_USER"
}
```

### CHECK USERS

```http
GET http://localhost:8080/users
```

### CREATE GROUP

```http
POST http://localhost:8080/groups
Content-Type: application/json

{
  "name": "Manali Trip"
}
```

### ADD USER TO GROUP

```http
POST http://localhost:8080/groups/1/users
Content-Type: application/json

[1, 2, 3]
```

### CHECK GROUP

```http
GET http://localhost:8080/groups
```

### CREATE EXPENSE

Run these as 2 separate requests:

```http
POST http://localhost:8080/expenses
Content-Type: application/json

{
  "description": "Bungee Jumping",
  "amount": 2700,
  "paidByUserId": 1,
  "groupId": 1
}
```

```http
POST http://localhost:8080/expenses
Content-Type: application/json

{
  "description": "Maggie",
  "amount": 200,
  "paidByUserId": 2,
  "groupId": 1
}
```

### VIEW ALL EXPENSES OF GROUP

```http
GET http://localhost:8080/expenses/group/1
```

### SHOW SETTLEMENT

```http
GET http://localhost:8080/expenses/settlements/1
```

### UPDATE EXPENSE

```http
PUT http://localhost:8080/expenses/1
Content-Type: application/json

{
  "description": "Bungee Jumping Updated",
  "amount": 3000,
  "paidByUserId": 2,
  "groupId": 1
}
```

### DELETE EXPENSE

```http
DELETE http://localhost:8080/expenses/2
```
"# SmartSplit" 
