```config.py``` acts as a single source of truth for runtime settings across the backend.
Rather than scattering magic numbers (like ```30``` days, ```20.0%``` interest) or sensitive strings(```SECRET_KEY```, database paths) across dozens of files, this file centralizes them into a strongly typed, validated configuration object

### Line-byLine Teachnical Breakdown
```python
import os
from pydantic_settings import BaseSettings
```
(import os): Python's standard library module to interact with the underlying operating system. it provide ``` os.getenv(), which reads environment variables set in your shell or Linux environment.
os: The operating system module
.getenv() A function that retrieves an environment variable
For examples: ```os.getenv("SECRET_KEY");```  This means that Python check whether the operating system has an environment variable called SECRET_KEY. If it exists, give me it value.

```from pydantic_settings import BaseSettings```In modern Pydantic (v2), application settings are handled by pydantic-settings. BaseSettings automatically reads environment variables, converts types (e.g., parsing a string "30" into a Python integer 30), and checks constraints.
Pydantic is a python library used for data validation and parsing; 
Pydantic helps ensure that the data entering your application has the correct structure and data types
Pydantic_settings is an additional package designed specifically for managing  application configuration and settings:
Remember
Pydantic: Validates and parses data
Pydantic Settings: Uses Pydantic capabilities to manage application settings,including loading them from the environment variables.
BaseSettings is a class provided by the pydantic-settings: it gives you a foundation for creating the settings class that automatically retrieves configuration values from environmental variables. Instead of manually retrieving every environment variable using os.getenv(), you can define a class:
```python
class Settings (Base Settings):
```
```python
PROJECT_NAME: str = "Loan Management System"### This is a Python Variable with a type annotation
## PROJECT_NAME: Variable name
## str: expected data type(string)
## "Loan Managment system" default value
#The colon followed by str is called a type hint.
CURRENCY: str = "KES"
```
```PROJECT_NAME```: Exposed via ```/api/health``` or system summaries so the frontend knows what system is talking to
```CURRENCY```: str = "KES": Sets Kenyan Shillings as the default monetary unit. When the frontend formats money ( eg; ```KES 10,OOO```), it conforms to this business standard

```python
 SECRET_KEY: str = os.getenv(
    "SECRET_KEY",
    "dev-secret-key-keith-lms-2026-super-secure"
 )
# SECRET_KEY is the variable that stores our application's secret key. Our application can access this variable whenever it needs to perform cryptographic operations
# str: Is a Python type annotation.: it tells developers and tools that SECRET_KEY is expected to contain a string, hence helps in making our code easier to undertsand
#  os.getenv(): means go and look for an environment variable  with this name and retrieve it's value: In our code above the second argument is a fallback value.
## It means that if SECRET_KEY exists in the environment, use it's value, and if it doesn't exist, use the default value


```
ALGORITHM
```python
ALGORITHM: str = "HS256"
# This specifies the cryptographic algorithm used to sign JWT tokens
#An algorithm is simply a set of mathematical instructions used to perform a particular task.
# In our authentication system the algorithm determines HOW our application will create and verify the digital signature on a JWT token.

# ALGORITHM work together SECRET_KEY, but they have different responsibilities 
# Think about it this way:
# SECRET_KEY: Is the secret ingredient.
# ALGORITHM is the mathematical method used with that ingredient
```
HS256 means:
HS = HMAC
256 = SHA-256
HMAC using SHA-256
HMAC stands for: Hash-based Message Authentication Code.:Explanation: Imagine you write a letter to someone. You want the receiver to know two things ```1. The letter genuinely came from you``` , ```2. Nobody changed the contents while the letter was being delivered.``` You and the receiver share a secret, You use that secret to create a special digital signature for the letter. The reciever uses the same secret to verify the signature. Thats the basic idea behind HMAC
In Your LMS: Your application creates the JWT signature, Your application uses the secret key, When the token comes back, your application verifies it's signature
Important: HMAC uses a shared secret. Both signing and verification use the same secret key

What is SHA-256?
SHA stands for Secure Hash Algorithm
SHA-256 is a cryptographic hashing algorithm.
it takes information of almost any size and produces a fixed-length hash of 256 bits
For example:
```python
import hashlib
message = "Hello LMS"
result = hashlib.sha256(
   message.encode()
).hexdigest

print(result)
# This produces a hexadecimal hash.
# A few important properties:
# 1. The same input produces the same hash
# 2. A small change in the input produces a different hash.
# 3. It is designed to be computationally infeasible to reverse the original input
```
Notice that SHA-256 produces a hash, but hashing alone does not prove who created the message.
That's why HMAC is useful: it combines hashing with a secret key

In simpler terms:
HMAC is a cryptographic signing mechanism
SHA-256 is a hashing algorithm
Together, they help ensure that a JWT hasn't been modified
Your application uses the SHA-256 hashing algorithm inside the HMAC mechanism. The result is a digital signature that depends on ``` The JWT's header``` , ``` The JWT payload``` , ``` Your SECRET_KEY```
Example:
```python
from jose import jwt
token = jwt.encode(
   {"sub": "user123"},
   settings.SECRET_KEY,
   algorithm=settings.ALGORITHM
)
```
```python
from jose import jwt
# This imports the JWT functionality from the python-jose library
#it gives the application tools for: Creating JWT tokens, Signing JWT tokens, Decoding JWT tokens , Verifying JWT signatures

```
```python
jwt.encode()
# This function creates a JWT token. Think of it as telling Python "Take this information, create a JWT, and sign it."
```
```python
{"sub": "user123"}
#This is the payload. The payload contains information about the token.
#In the code above:
#sub stands for Subject:It identifies who the token is about. In our LMS it might represent a student, lecturer, teacher, businessman
#For example {"sub": "25" } This could mean that the token belongs to the user whose database ID is 25.
# Notice that sub is a standard JWT claim.
```
```python
settings.SECRET_KEY
#This supplies the secret key we discussed before
#it is the secret used to generate the JWT signature
```
```python
algorithm=settings.ALGORITHM
# This tells the JWT library which signing algorithm to use
#In our case:
ALGORITHM = "HS256"
#Therefore our application uses HMAC with SHA-256
```

How does our application verify the Token?
Creating the token is only half of the process. Our application must also verify it when a user makes another request.
For example:
```python
from jose import jwt
payload = jwt.decode(
   token,
   settings.SECRET_KEY
   algorithm=[settings.ALGORITHM]
#Notice something interesting:
#When encoding, we write:
algorithm=settings.ALGORITHM
# When decoding we write:
algorithm=[settings.ALGORITHM]
# The decoding function accepts a list of permitted algorithms. Our application use this to verify the token's signature.
# If verification fails, an exception is raised.
#In a real authentication system, you should alsp validate token expiration and claims, and handle invalid-token NOTE:
)
```

```python
ACCESS_TOKEN_EXPIRES_MINUTES: int = 60 * 24
#This determines how long an authentication token remains valid.
#Understanding the calculations:
#60 seconds? No, 60 minutes=1hour
#24 hours= 24*60 minutes
# Therefore 60 * 24 = 1,440 minutes.
# The importance of this is that whe a user logs in, the application uses an access token. That token expires after 24 hours, meaning the user must authenticate again or use a refresh-token mechanism to obtain another access token.
```
```python
DATABASE_URL: str = os.getenv(
   "DATABASE_URL",
   "sqlite:///./data/lms.db"
# This tells our application where it's database is located and which database engine to use
# sqlite: database engine
# ./:current project directory
# data/: directory containing the database
# lms.db: database file.

)
``` 
```python
# If later on you want to change to PostgreSQL instead, then you could change the environment variable:
DATABASE_URL=postgresql://user:password@localhost:5432/lms
# Our application can then use PostgreSQL instead of SQLite, provided our database connection code and dependencies support it.
#This is one major advantage of configuration management: you can change environments without rewriting our application logic.
```

```python
DEFAULT_LOAN_DURATION_DAYS: int = 30
# This defines the default repayment  duration for a loan
#We could use it like this:
from datetime import date, timedelta

loan_date = date.today()

due_date = loan_date + timedelta(
   days=settings.DEFAULT_LOAN_DURATION_DAYS
   
)
#The result is payment due date 30 days after the loan date.
```

```python
DEFAULT_INTEREST_RATE_PERCENT: float = 20.0
#This defines the default interest rate
# Interest = Principal * Rate
#NOTE: For actual financial calculations, consider using Python's Decimal type instead of float to avoid floating-point precision issues
```
```python
TIMEZONE: str = "Africa/Nairobi"
# This defines the timezone that our application should use when dealing with dates and times

# This matters because if your server for example is hosted in London and your customers are in Kenya, your server uses UK time, and Your customers expect loan due dates and transaction times in Kenyan time.
# The timezone setting tells our application which timezone to use when displaying or interpreting local times
# Example
from zoneinfo import ZoneInfo
from datetime import datetime

now = datetime.now(
   ZoneInfo(settings.TIMEZONE)
)
print(now)
# This produces the current date and time in Nairobi
```


### Understanding the Config class
```python 
class Config:
   env_file = ".env"

# This is a configuration class telling Pydantic Settings where to find environment variables
```
An .env file is simply a file containing configuration values
Example:
```env
PROJECT_NAME=Loan Management System
CURRENCY=KES
SECRET_KEY=my-random-secret
DATABASE_URL=sqlite:///./data/lms.db
DEFAULT_LOAN_DURATION_DAYS=30
DEFAULT_INTEREST_RATE_PERCENT=20.0
```
When our application starts, Pydantic Settings reads the .env file and loads the values



What is a .env file?
A ```.env``` file is a file commonly used to store configuration values for our application.
For example inside your .env file you might have something like this: ``` SECRET_KEY=my-very-long-random-secret``` Hnece your Python application can now retrieve that value
However , there is an Important detail: Python's os.getenv() does not automatically read a .env file. You need a library such as python-dotenv to load it.
You will have something like this:
```python
import os
from dotenv import load_dotenv
load_dotenv()
SECRET_KEY: str = os.getenv("SECRET_KEY")
# Now Python can read the value from your .env file.
```
HOW SECRET_KEY PROTECTS JWT TOKENS: Explanation: Imagine a student named John Logs into our LMS: The process looks like this: John enters his username and password, our application verifies his credentials, then our application creates a JWT, then our application signs the JWT using the secret key, The token is sent to John's browser, John uses that token when accessing protected LMS resources. Our application verifies the token's signature using the secret key. If someone modifies the token, the signature verification should fail.

NOTE: Important: JWT signatures provide integrity and authenticity, not encryption. JWT payloads are often readable, so don't put passwords or other sensitive information inside them.

You can generate a strong secret using Python:
```python
import secrets
secret = secrets.token_urlsafe(64)
print(secret)
# This generates cryptographically secure random secret
# NOTE: Never expose the actual production secret in your source code or public Github repository
```
### NOTE:
```
SECRET_KEY is a secret used for cryptographic operations such as signing JWTs.

os.getenv() retrieves configuration from environment variables.

.env helps you keep secrets outside your source code, but you must load it.

One more distinction worth remembering: SECRET_KEY is not the same as a user's password. A user's password authenticates that user; your application's secret key protects cryptographic operations performed by the application.
```

### Understanding setting=Settings()
```python
settings = Settings()
# Defining a class doesn't automatically create an object, we need to instantiate it:
# In the above code student is an object created from the Student class. And once we create an instance of our Settings class, Pydantic Settings loads and validates the configuration values
#Now we can access our settings anywhere in our application
```
