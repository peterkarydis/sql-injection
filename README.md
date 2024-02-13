# SQL-Injection (Seed Security Labs)

**Running:** SEEDUbuntu16.04 VM

## Vulnerability description
- User input is not properly sanitized by the PHP web app, allowing an attacker to inject SQL code into a query and gain unauthorized access to the application's database and/or perform unauthorized actions.

## Login to DB as admin
- Scenario in which we know the username "admin" and not the password
- Vulnerable PHP code:
![unsafe_home.php](https://github.com/peterkarydis/sql-injection/blob/main/images/pic1.png?raw=true)

- The attacker can login as admin when giving the following username input:
![admin_login](https://github.com/peterkarydis/sql-injection/blob/main/images/pic2.png?raw=true)

- The attacker tricks the database server into returning a select query without asking for the password, due to commenting out everything after the # symbol given to the username input field.
![admin_user_details](https://github.com/peterkarydis/sql-injection/blob/main/images/pic3.png?raw=true)

## Login to DB as admin using curl
```
curl 'www.SeedLabSQLInjection.com/unsafe_home.php?username=admin%27%3B%23'
```
- %27, %3B, %23 are the html url encoded ' ; and # symbols respectively.
- This curl command returns the admins' info. Part of the output is the following:
![admin_user_details_curl](https://github.com/peterkarydis/sql-injection/blob/main/images/pic4.png?raw=true)

## Update/Delete from DB
- An attacker could also modify DB data through converting an sql statement to two, by again exploiting the vulnerable login php code.We can seperate two SQL statements by using the semicolon (;).

```
'1=1;delete from credential where name='Ted';#
```
- The following attack does not work against MySQL because the mysqli PHP extension "mysqli :: query () API" does not allow multiple queries to be executed to the DB server. If we would like to run multiple SQL commands, we could use $ mysqli -> multi_query (), which is not recommended.

## DB user changes values he is not authorized to
- In this scenario we login as Alice, and change our salary (which we are normally not able to)
- Vulnerable php code for users to edit their profile:
![unsafe_edit_frontend.php](https://github.com/peterkarydis/sql-injection/blob/main/images/pic5.png?raw=true)

- Attacker can exploit this vulnerability by giving the following input:
```
',salary=50000 where EID=10000;#
```
![alice_changes_her_salary](https://github.com/peterkarydis/sql-injection/blob/main/images/pic6.png?raw=true)

## DB user changes password of other user
- First we generate the hash value of the new password, because this will be saved inside the DB.
![newpassword_hash](https://github.com/peterkarydis/sql-injection/blob/main/images/pic7.png?raw=true)

- Then we login as Alice and edit our profile (similarly to previous injection):
```
',password='f2c57870308dc87f432e5912d4de6f8e322721ba' where name='Boby';#
```

## SQL Injection Countermeasures
- Using prepared statements.
- PHP Code using prepared statement:
![safe_home.php](https://github.com/peterkarydis/sql-injection/blob/main/images/pic8.png?raw=true)

- Using prepared statement means taking two steps into sending the SQL statement to the DB.
1. Only send the data part, without the real data. The real data are replaced by (?).
2. Send the data to the DB using bind_param(). Through this method the DB takes the data as data and not code.
- Through prepared statements the DB has a clear distinction between **code** and **data**.
- The attacker could hide code into the data, but the code will never be interpreted as code, which means it will never execute.