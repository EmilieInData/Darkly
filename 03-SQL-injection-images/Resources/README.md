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

On the **Searchimg** page, there is a form that allows users to search for images by ID.

From the previous SQL injection, which returned all tables and their columns, we discovered that there is a table called `list_images` containing four fields: `id`, `url`, `title`, and `comment`.

We can therefore try the following injection:

```sql
1 UNION SELECT title, comment FROM list_images
```
This query successfully returns the contents of the title and comment fields. One of the comments contains an encrypted message that we must decrypt and then re-encrypt according to the challenge instructions in order to obtain the flag.

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

