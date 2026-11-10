This file is responsible of connecting our FASTAPI application to a database and managing how the application communicates with the database.
It handles five main things:
```bash
  1.Database connections: Establishes a connection between our application and the database.
  2. SQLite configuration: Configures SQLite to work better with FASTAPI's threading model
  3. Database sessions: Create sessions through which our application interacts with the database.
  4. Database models: Creates a base class that other database models will inherit from.
  5 Database dependency: Provides database sessions to our FASTAPI endpoints and ensures they are closed afterwards.
  ```

Our database.py file prepares everything our backend needs to communicate with the database safely, consistently, and efficiently.

ORM( Object-Relational Mapping) is one of the most important concepts to understand before going further.
 When we break down the name:
 ```bash
  1.Object:Refers to Python objects and classes.
  2.Relational: Refers to relational databases. which organize information into tables
  3. Mapping: Means establishing a relationship between the two.
  ```
In simple terms ORM allows us to represent database tables using Python classes.
The Important thing to understand: ORM creates a bridge between how Python represents information and how relational databases store information.


Line-by-LIne Technical Breakdown:
imports:
```python
from sqlalchemy import create_engine, event
from sqlalchemy.orm import declarative_base, sessionmaker
from app.config import settings
```
1.SQLAlchemy is a Python library that allows our application to interact with databases using Python code instead of writing raw SQL queries for every operation. It acts as a bridge between our Python application(eg FastAPI) and a database(e.g SQLite, PostgreSQL, or MySQL)
What does SQLAlchemy do?
```bash
 1.Connects our Python application to a database.
 2.Create database tables and defines their structure.
 3.Performs CRUD operations: Create, Read, Update, and   Delete records
 4.Manage database connections and sessions.
 5.Translate Python operations into SQL queries.
 ```

***`2. create_engine`***: Creates the core interface between Python and the underlying SQLite database driver. It manages connections and dialect translations.
```python
# What is an engine?
# An engine is SQLAlchemy's central interface for communicating with a database
# Our application needs a way to communicate with SQLite. The SQLAlchemy engine provides that connection infrastructure.
#For example:
engine = create_engine(
    "sqlite:///./database.db"
)
#create_engine() is a function provided by SQLAlchemy.
#sqlite:///./database.db is the database connection URL.
#engine is the object returned by that function.

#The engine knows: which database we are connecting to. which database drivers to use, How to establish database connections, How to translate SQLAlchemy operations into database-specific SQL.
#Think of an engine as the communication gateway between our Python application and the database.
# NOTE: One important distinction: creating an engine doesn't necessarily mean that a database connection has already been established. SQLAlchemy generally establishes connections when they're first needed.
```

***`3. event `***: SQLAlchemy's event hook system. It allows running custom logic at specific database lifecycle hooks, such as when a new low-level connection is opened.
```python
# An event is something that happens during the execution of a program
# For example:
# 1. A user registers
# 2. Someone logs in
# 3. A database connection is created
# 4. A database transaction begins
# SQLAlchemy has an event system that allows us to execute our Python functions when particular database events occur
# For example:
from sqlalchemy import event

@event.listens_for(engine. "connect")
def receive_connect(dbapi_connection, connection_record):
    print("A new database connection was created!")
#event.listens_for(): registers a function to listen to a particular event.
#engine: Identifies the SQLAlchemy engine to monitor.
#connect: Identifies the event we're interested in.
#receive_connect() is the function that executes when a new database connection is created.

@event.listens_for(engine, "connect")
def enable_foreign_keys(dbapi_connection, connection_record):
    cursor = dbapi_connection.cursor()
    cursor.execute("PRAGMA foreign_keys=ON")
    cursor.close()
#For example the code above ensures that SQLite enables foreign key enforcement on newly created connections
#Think of event as a notification system that let's us execute custom instructions when something happens inside SQLAlchemy.

```

***`4. declarative_base`***: A factory function that constructs a base class. When our models inherit from this base class, SQLAlchemy inspects their Python attributes and converts them into relational table columns.
This one is particularly important because it helps us define database tables using Python class
```sql
/* Normally, when working directly with SQL, you create tables like this:*/
CREATE TABLE users(
    id INTEGER PRIMARY KEY.
    username VARCHAR(59),
    email VARCHAR(100)
)
```
But SQLAlchemy allows us to represent the table above using a Python class as shown below:
```python
Base = declarative_base()## This creates a special base class. We use this class as the parent of our database model class

class User(Base):## This means the User class inherits from Base. And by inheriting from Base, SQLAlchemy knows that this class is intended to represent a database model.
    __tablename__ = "users"## This tells SQLAlchemy the name of the database table represented by this class

#The codes below define columns of the database table
    id = Column(Integer, primary_key=True)
    username =Column(String(50))
    email = Column(String(100))

# The result of all these is a Python class that SQLAlchemy maps to a relational database table.

#NOTE: Think of declarative_base as the foundation upon which we build all our database models
```

***`5. session_maker`***: A factory pattern that generates new Session objects with predefined configutations
```python
# This concept is extremely important when building FastAPI applications.
#First we have to understand what a database session is.
# A database session is an object that SQLAlchemy uses to manage our interactions with the database.
# For example, suppose a user registers on our application. Our application needs to 
# 1. Open a database session
# 2. Add the new user.
# 3. Commit the transaction
# 4. Close the session.
# Below is an example

db = SessionLocal()

new_user = User(
    username="john",
    email="john@gmail.com"
)
db.add(new_user)
db.commit()
db.close()

# But where does SessionLocal from?
# That's where the sessionmaker enters the picture
SessionLocal = sessionmaker(
    autocommit=False,
    autoflush=False,
    bind=engine,
)
#autocommit=False: Don't automatically commit transactions
#autoflush=False: Don't automatically flush pending changes before queries.
#bind=engine: Associate the session with the database engine.

# NOTE: The important distinction:
# sessionmaker(): creates a factory for database sessions
#SessionLocal() creates an actual database session

#Think of sessionmaker as a machine that produces database sessions whenever our application needs one
```

***`from app.config import settings`*** This line is importing configuration information from another part of our application
```python
# Inside config.py we have
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    DATABASE_URL: str = os.getenv("DATABASE_URL", "sqlite:///./data/lms.db")

settings = Settings()


# Now Inside database.py:
from app.config import settings
# We can access the database URL using:
settings.DATABASE_URL
#And pass it to SQLAlchemy:
engine = create_engine(
    settings.DATABASE_URL
)
# We do this because we don't want to hardcode database connection details everywhere in our application. Instead we centralize our configurations in one place.

#For example, we could change our database from SQLite to PostgreSQL by changing the database URL, without having to rewrite all our database connection code.
```


