# SQL Injection

## What is SQL Injection?

SQL Injection is a vulnerability that allows an attacker to modify an SQL statement through unsanitized user input, such as a form field.

By injecting SQL code into a query, an attacker can alter its behavior and gain access to data that should not be exposed. This can lead to the disclosure of database contents, unauthorized access to sensitive information, or the collection of information about the database structure through error messages.

For example, in a MariaDB database, the following query can be used to test whether an SQL injection is possible:

```sql
SELECT * FROM users WHERE id = 1 OR 1=1;
```

Since the condition `1=1` is always true, the query returns all records from the table.

## How to Find the Vulnerability?

On the **Members** page, there is a form that allows users to search for members by ID.

First, we can determine whether SQL injection is possible and identify the type of database by performing a blind test:

```sql
1' OR '1'='1
```

This input returns a syntax error indicating that the backend uses **MariaDB**, confirming that user input is not properly sanitized.

Since the form is vulnerable, we can inject SQL statements.

We can also infer that the original query contains two selected columns:

```sql
SELECT <param1>, <param2> FROM users WHERE id =
```

Using the `UNION` operator, we can combine the original query with another one:

```sql
1 UNION SELECT table_name, column_name FROM information_schema.columns
```

This query returns the names of tables and columns stored in the database metadata, allowing us to discover the database structure.

Once the available columns have been identified, we can extract data from them:

```sql
1 UNION SELECT Commentaire, countersign FROM users
```

This reveals the contents of the selected fields. In this challenge, one of these fields contains the flag that must be decrypted and re-encrypted to complete the exercise.

## Suggestions to Prevent SQL Injection

### Use Prepared Statements

Prepared statements separate SQL code from user input and prevent injected code from being executed.

```sql
SELECT * FROM users WHERE id = ?;
```

### Validate and Sanitize User Input

Only allow the expected type of data. For example, if a field expects an integer, reject any non-numeric characters.

### Hide Database Errors

Do not display detailed SQL error messages to users. Instead, return generic error messages.

### Apply the Principle of Least Privilege

Database accounts should only have the permissions required to perform their tasks.

### Use an ORM or Query Builder

Modern ORMs and query builders automatically handle parameter binding and help prevent SQL injection vulnerabilities.

