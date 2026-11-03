# SQL Injection

## Description

SQL Injection occurs when untrusted user input is included in a SQL query without proper validation or parameterization.

## Example

Vulnerable code:

```sql
SELECT * FROM users WHERE username = '$username'
```

An attacker may inject:

```sql
' OR '1'='1
```

## Risks

- Unauthorized data access
- Data modification
- Data deletion
- Authentication bypass

## Mitigation

- Parameterized queries
- Prepared statements
- Input validation
- Least privilege database accounts

## OWASP Category

A03:2021 - Injection
