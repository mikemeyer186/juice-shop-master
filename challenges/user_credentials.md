# OWASP Juice Shop challenge: User Credentials

## About the challenge

|            |                                        |
| ---------- | -------------------------------------- |
| Titel      | User Credentials                       |
| Category   | SQL Injection                          |
| Task       | Retrive a list of all user credentials |
| Difficulty | ⭐️⭐️⭐️⭐️                           |

### Description

In this challenge this task is to find a possibility to get all the user data from the database of Juice Shop. To achieve this huge data breach it is necessary to find an vulnerable endpoint where data can be retrieved from the server and by passing a SQL injection. A good starting challenge is “Exfiltrate the entire Database schema definition via SQL Injection”.

> [!NOTE]
> You have to use a software tool to intercept and manipulate the HTTP requests (e.g. Burp Suite) in your local environment.

<br>

## Approach

### 1. Identify which enpoint can be used to inject the SQL code

For this SQL injection you need a HTTP request to get data from the server and where the parameter is processed on the server. For our purpose the product api is the vulnerable enpoint, because the parameter (q = product to search) will be processed server-side and not client-side. You can easily find this rest api by executing the product search on the start page and see the corresponding GET request in the HTTP traffic history.

```
GET /rest/products/search?q=<search string>      # returns a JSON object with the search results
```

<br>

### 2. Manipulate the GET request and its parameter

You have to manipulate the GET request with a UNION SELECT query to get the user credentials from the 'Users' table from the database. It is necessary that you know the database type and schema to query the correct table and fields. A good way is to solve the challenge “Exfiltrate the entire Database schema definition via SQL Injection” before.

```
# returns the sqlite schema
GET /rest/products/search?q=apple'))UNION%20SELECT%20sql,2,3,4,5,6,7,8,9%20FROM%20sqlite_master--

# returns the username, email and password from table 'Users'
GET /rest/products/search?q=apple'))UNION%20SELECT%20username,email,password,4,5,6,7,8,9%20FROM%20Users--
```

<br>

## Explaination

### Vulnerability

The endpoint '/rest/products/search?q=' is vulnerable to SQL Injection because the value of the 'q' parameter is processed on the server to return the corresponding search results. If the search were implemented client-side — meaning all products would be loaded first and then filtered — this type of attack would not be possible. However, in this case, the search is performed on the server, and only products matching the parameter are returned. By gradually modifying the parameter, meaningful error messages can be obtained, providing clues about the structure of the SQL query. An UNION SELECT query is used to append the injected query to the regular search query. To retrieve user data, knowledge of the database schema is required. The 'sql' field can be queried from the 'sqlite_master' table, which reveals that there is a 'Users' table containing fields such as username, email, and password. The UNION SELECT query can then be adjusted accordingly to extract user data for all users from the database. Even the password will be retrieved, but converted to a hash string.

### Conseqences

In a real-world scenario, this vulnerability could cause massive damage to the shop and to the users. Such a data breach can lead to a loss of personal data, like e-mail addresses and passwords. Attackers can use them to get access to other personal accounts even on other websites (e.g. banking, email). If these user credetials were leaked (e.g. on Darknet platforms), these user data can be used for online frauds, phishing attacks or even identity theft. For the shop this attack means a huge financial loss due to legal penalties (e.g. DGSVO, GDPR) or compensation claims from affacted users. The image of the shop will be damaged as well due to customer loss and maybe negative press on social media. Such a data breach will lead to an investigation through authorities. To stop further damage the shop may need to be taken offline for a while to fix the vulnerability by IT department.

### How to avoid SQL Injections?

This vulnerability can be prevented by using prepared statements with bound paramters. That ensures that the user input is never executed as SQL code. It is also a good way to validate the user input. Unexpected input should be rejected by the server. This can be achieved with withelisting. A closer look on the permissions is required as well. The DB user should have the minimum necessary permissions. It is also a good idea to use generic error messages that never exposes raw database errors and shows too much information about the DB framework. One good point can be a pentesting to see possible vulnerabilites before a criminal hacker can do that. Also a detection of SQL Injection attempts in server logs can show a anomalous behavior in HTTP requests.

<br>

## Video Documentation (in German language)

[![](/challenges/assets/user_credentials.png)](https://www.loom.com/share/da8bbe10e6044871ac3568ab203d8670)
