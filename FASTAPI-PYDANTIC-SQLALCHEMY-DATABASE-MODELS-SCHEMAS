# FastAPI, Pydantic and SQLAlchemy: Complete Database Models and Schemas Guide

## Table of Contents

1. Understanding the architecture
2. Project structure
3. Installing dependencies
4. Understanding database.py
5. SQLAlchemy Engine
6. Database URLs
7. SessionLocal and sessionmaker
8. DeclarativeBase
9. Understanding get_db()
10. Understanding SQLAlchemy models
11. SQLAlchemy 2.0 typed mappings
12. Understanding mapped_column()
13. Understanding User model
14. Understanding Post model
15. Understanding ForeignKey
16. Understanding relationship()
17. Understanding back_populates
18. Understanding from __future__ import annotations
19. Understanding Pydantic schemas
20. Why separate schemas from models?
21. BaseModel
22. Field() validation
23. ConfigDict and from_attributes
24. Create and response schemas
25. Optional fields and defaults
26. Connecting Pydantic to SQLAlchemy
27. Creating database tables
28. Database queries
29. Creating posts
30. Retrieving posts
31. Retrieving user posts
32. Updating posts
33. Deleting posts
34. SQLAlchemy relationship loading
35. HTML templates
36. Complete implementation
37. Testing the API
38. Error handling
39. Database transactions
40. SQLAlchemy vs Django ORM
41. Common mistakes
42. Exercises
43. Quick reference

---

# SECTION 1: Understanding the Architecture

When building a FastAPI application with SQLAlchemy and Pydantic, you should understand that these technologies have different responsibilities.

You normally separate your application into three important files:

- `database.py`
- `models.py`
- `schemas.py`

You then use these files inside your route functions.

## 1.1 database.py

This file is responsible for database connectivity.

It normally contains:

- Database URL
- SQLAlchemy engine
- Session factory
- Declarative base class
- Database dependency

## 1.2 models.py

This file contains SQLAlchemy ORM models.

These models describe the structure of your database tables.

For example:

```python
class User(Base):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(primary_key=True)
    username: Mapped[str] = mapped_column(String(50))
```

This describes a database table called `users`.

## 1.3 schemas.py

This file contains Pydantic models.

These models define the data your API accepts and returns.

For example:

```python
class UserCreate(BaseModel):
    username: str = Field(min_length=3, max_length=50)
    email: EmailStr
```

This validates incoming user data.

## 1.4 How everything works together

Imagine a client sends this JSON:

```json
{
    "title": "Learning FastAPI",
    "content": "Today I learned SQLAlchemy and Pydantic.",
    "user_id": 1
}
```

The process looks like this:

```text
             CLIENT
                |
                |
                v
          JSON REQUEST
                |
                v
       PYDANTIC SCHEMA
       (schemas.py)
                |
                | Validates data
                v
          FASTAPI ROUTE
                |
                | Uses validated data
                v
       SQLALCHEMY MODEL
        (models.py)
                |
                | Creates ORM object
                v
        DATABASE SESSION
        (database.py)
                |
                | Executes SQL
                v
            DATABASE
                |
                | Returns database data
                v
        SQLALCHEMY OBJECT
                |
                v
       PYDANTIC RESPONSE SCHEMA
                |
                | Serializes data
                v
           JSON RESPONSE
```

Remember:

**Pydantic validates data. SQLAlchemy models represent database tables. SQLAlchemy sessions communicate with the database.**

---

# SECTION 2: Project Structure

A simple project structure:

```text
blog_api/
│
├── app/
│   ├── __init__.py
│   ├── main.py
│   ├── database.py
│   ├── models.py
│   ├── schemas.py
│   │
│   └── routers/
│       ├── __init__.py
│       ├── posts.py
│       └── users.py
│
├── blog.db
│
└── pyproject.toml
```

For a beginner project, you can keep your routes inside `main.py`.

As the project grows, separate your routes into routers.

---

# SECTION 3: Installing Dependencies

Using UV:

```bash
uv init
uv add fastapi "uvicorn[standard]" sqlalchemy pydantic
```

If you want to use email validation:

```bash
uv add email-validator
```

If you want PostgreSQL:

```bash
uv add "psycopg[binary]"
```

Start your application:

```bash
uv run uvicorn main:app --reload
```

If your main.py is inside the app directory:

```bash
uv run uvicorn app.main:app --reload
```

---

# SECTION 4: Understanding database.py

Let's examine the database configuration.

```python
from sqlalchemy import create_engine
from sqlalchemy.orm import DeclarativeBase, sessionmaker

SQLALCHEMY_DATABASE_URL = "sqlite:///./blog.db"

engine = create_engine(
    SQLALCHEMY_DATABASE_URL,
    connect_args={"check_same_thread": False},
)

SessionLocal = sessionmaker(
    autocommit=False,
    autoflush=False,
    bind=engine,
)


class Base(DeclarativeBase):
    pass


def get_db():
    with SessionLocal() as db:
        yield db
```

We will examine every important part.

## 4.1 What is SQLAlchemy?

SQLAlchemy is a Python SQL toolkit and Object Relational Mapper (ORM).

An ORM allows you to interact with database records using Python objects rather than manually writing every SQL statement.

For example, instead of:

```sql
SELECT * FROM users WHERE id = 1;
```

You can write:

```python
user = db.get(User, 1)
```

Instead of manually writing an INSERT statement:

```sql
INSERT INTO users (username, email)
VALUES ('nzegge', 'user@example.com');
```

You can write:

```python
user = User(
    username="nzegge",
    email="user@example.com",
)

db.add(user)
db.commit()
```

SQLAlchemy translates ORM operations into SQL statements.

## 4.2 What is create_engine()?

```python
from sqlalchemy import create_engine

engine = create_engine(
    SQLALCHEMY_DATABASE_URL,
)
```

`create_engine()` creates an SQLAlchemy Engine.

The engine is responsible for managing database connectivity.

Think of the engine as the central connection manager between your Python application and the database.

It manages connections and connection pooling.

It is not the same thing as a database session.

### Important distinctions

| Object | Responsibility |
|---|---|
| Engine | Manages database connectivity |
| Connection | Represents a database connection |
| Session | Manages ORM operations and transactions |
| ORM model | Represents a database-mapped Python object |

You normally create one engine for your application process, rather than creating a new engine for every request.

## 4.3 Understanding the database URL

```python
SQLALCHEMY_DATABASE_URL = "sqlite:///./blog.db"
```

This is a SQLAlchemy database URL.

For SQLite:

```python
sqlite:///./blog.db
```

means a SQLite database file named `blog.db` in the current working directory.

Other examples:

```python
sqlite:///./database.db
```

```python
postgresql+psycopg://username:password@localhost:5432/blog_db
```

```python
mysql+pymysql://username:password@localhost:3306/blog_db
```

The URL tells SQLAlchemy:

- Which database system to use
- Which driver to use
- Where the database is located
- Which credentials to use when required

Do not hardcode real database passwords in production code. Use environment variables.

## 4.4 Understanding check_same_thread=False

```python
engine = create_engine(
    SQLALCHEMY_DATABASE_URL,
    connect_args={"check_same_thread": False},
)
```

This option is specific to SQLite's Python driver.

SQLite normally restricts a connection to the thread that created it.

FastAPI may execute synchronous endpoint code and dependencies in worker threads.

Setting:

```python
"check_same_thread": False
```

allows the SQLite connection to be used from different threads.

Important:

- It does not make SQLite a high-concurrency database.
- It does not make SQLAlchemy sessions safe for concurrent use.
- Each request should have its own session.
- Do not share one session between multiple requests.

You generally do not use this SQLite-specific option with PostgreSQL.

---

# SECTION 5: Understanding sessionmaker()

Your code:

```python
SessionLocal = sessionmaker(
    autocommit=False,
    autoflush=False,
    bind=engine,
)
```

First, understand what a session is.

A SQLAlchemy Session is an object that manages ORM database operations.

For example:

```python
db = SessionLocal()
```

This creates a new database session.

You can use the session to:

- Retrieve records
- Insert records
- Update records
- Delete records
- Commit transactions
- Roll back failed transactions

## 5.1 What is sessionmaker()?

`sessionmaker()` creates a factory for database sessions.

Think about it this way:

```python
SessionLocal = sessionmaker(bind=engine)
```

You have created a session factory.

Whenever you call:

```python
db = SessionLocal()
```

you receive a new session.

`SessionLocal` is not itself a session. It is the object that creates sessions.

## 5.2 Understanding bind=engine

```python
bind=engine
```

This tells the session factory which database engine to use.

Without the correct binding, the session would not know which configured database engine it should communicate with.

## 5.3 Understanding autocommit=False

```python
autocommit=False
```

This is the normal configuration for modern SQLAlchemy ORM sessions.

You explicitly control successful database writes using:

```python
db.commit()
```

For example:

```python
user = User(username="nzegge")

db.add(user)
db.commit()
```

The commit makes the transaction permanent.

## 5.4 Understanding autoflush=False

```python
autoflush=False
```

When autoflush is enabled, SQLAlchemy can automatically send pending changes to the database before certain queries.

Disabling it means SQLAlchemy will not automatically flush pending changes before every query.

You can still manually flush:

```python
db.flush()
```

Remember the difference:

```python
db.flush()
```

Sends pending SQL statements to the database inside the current transaction.

```python
db.commit()
```

Commits the transaction.

A flush is not a commit.

---

# SECTION 6: Understanding DeclarativeBase

Your code:

```python
from sqlalchemy.orm import DeclarativeBase

class Base(DeclarativeBase):
    pass
```

This class is the foundation of your SQLAlchemy models.

When you create a model:

```python
class User(Base):
    __tablename__ = "users"
```

SQLAlchemy recognizes it as a declarative ORM model.

The class inherits from `Base`, which provides access to SQLAlchemy's declarative mapping system and metadata.

You can access the registered table definitions through:

```python
Base.metadata
```

For a small project, you can create tables with:

```python
Base.metadata.create_all(bind=engine)
```

This creates tables that do not already exist.

Important: `create_all()` does not automatically alter existing tables when your models change.

For schema changes in a serious application, use Alembic migrations.

---

# SECTION 7: Understanding get_db()

Your code:

```python
def get_db():
    with SessionLocal() as db:
        yield db
```

This is a very important part of FastAPI dependency injection.

Let's break it down.

## 7.1 What does SessionLocal() do?

```python
SessionLocal()
```

Creates a new database session.

## 7.2 What does with do?

```python
with SessionLocal() as db:
```

The context manager manages the session's lifetime.

When the context exits, the session is closed.

## 7.3 What does yield do?

```python
yield db
```

`yield` temporarily provides the database session to the endpoint.

FastAPI recognizes this as a dependency with cleanup.

The process:

1. FastAPI calls `get_db()`.
2. A session is created.
3. `yield db` gives the session to the endpoint.
4. The endpoint performs its database operations.
5. FastAPI finishes the dependency lifecycle.
6. The context manager closes the session.

A typed version:

```python
from collections.abc import Generator
from sqlalchemy.orm import Session

def get_db() -> Generator[Session, None, None]:
    with SessionLocal() as db:
        yield db
```

## 7.4 Using get_db() inside an endpoint

```python
from typing import Annotated

from fastapi import Depends
from sqlalchemy.orm import Session

@app.get("/users")
def get_users(
    db: Annotated[Session, Depends(get_db)]
):
    ...
```

Let's understand:

```python
Depends(get_db)
```

This tells FastAPI to execute `get_db()` and provide its result.

The endpoint receives the session through:

```python
db
```

You do not manually call `get_db()` inside the endpoint.

## 7.5 Creating a reusable database dependency

Instead of repeating this:

```python
db: Annotated[Session, Depends(get_db)]
```

You can define:

```python
DatabaseSession = Annotated[
    Session,
    Depends(get_db),
]
```

Then:

```python
@app.get("/users")
def get_users(db: DatabaseSession):
    ...
```

This makes your endpoint signatures cleaner.

---

# SECTION 8: Understanding SQLAlchemy Models

Now let's examine `models.py`.

A SQLAlchemy model is a Python class mapped to a database table.

For example:

```python
class User(Base):
    __tablename__ = "users"
```

This creates a mapping between the Python `User` class and the database table named `users`.

## 8.1 SQLAlchemy imports

```python
from sqlalchemy import (
    DateTime,
    ForeignKey,
    Integer,
    String,
    Text,
)

from sqlalchemy.orm import (
    Mapped,
    mapped_column,
    relationship,
)
```

Let's understand them.

| Import | Purpose |
|---|---|
| `DateTime` | Date and time database column |
| `ForeignKey` | Defines a reference to another table |
| `Integer` | Integer database column |
| `String` | String database column |
| `Text` | Text database column |
| `Mapped` | SQLAlchemy typed mapping annotation |
| `mapped_column` | Defines ORM database columns |
| `relationship` | Defines ORM relationships between models |

---

# SECTION 9: Understanding Mapped and mapped_column()

Your code:

```python
id: Mapped[int] = mapped_column(
    Integer,
    primary_key=True,
    index=True,
)
```

This is SQLAlchemy 2.0's typed declarative style.

Let's examine the two important parts.

## 9.1 Mapped[int]

```python
id: Mapped[int]
```

`Mapped` tells SQLAlchemy that this attribute is mapped by the ORM.

The `int` indicates the expected Python-side type.

Other examples:

```python
username: Mapped[str]
```

```python
age: Mapped[int]
```

```python
price: Mapped[float]
```

```python
image_file: Mapped[str | None]
```

## 9.2 mapped_column()

```python
mapped_column(Integer, primary_key=True)
```

This declares the actual database column configuration.

For example:

```python
id: Mapped[int] = mapped_column(
    Integer,
    primary_key=True,
)
```

- `Integer` defines the database column type.
- `primary_key=True` defines the primary key.

Another example:

```python
username: Mapped[str] = mapped_column(
    String(50),
    nullable=False,
    unique=True,
)
```

This creates a string column with a maximum declared length of 50 and database constraints.

## 9.3 Understanding primary_key=True

```python
id: Mapped[int] = mapped_column(
    Integer,
    primary_key=True,
)
```

A primary key uniquely identifies a database row.

For example:

| id | username |
|---|---|
| 1 | alice |
| 2 | bob |
| 3 | charles |

Each row has a unique ID.

## 9.4 Understanding index=True

```python
id: Mapped[int] = mapped_column(
    Integer,
    primary_key=True,
    index=True,
)
```

An index helps the database find records efficiently.

Primary keys are already indexed by normal database implementations, so explicitly adding `index=True` to a primary-key column is generally unnecessary.

For a foreign key that you frequently filter on, an index may be useful:

```python
user_id: Mapped[int] = mapped_column(
    ForeignKey("users.id"),
    index=True,
)
```

---

# SECTION 10: Understanding the User Model

```python
from __future__ import annotations

from sqlalchemy import Integer, String
from sqlalchemy.orm import Mapped, mapped_column, relationship

class User(Base):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(
        Integer,
        primary_key=True,
    )

    username: Mapped[str] = mapped_column(
        String(50),
        unique=True,
        nullable=False,
    )

    email: Mapped[str] = mapped_column(
        String(120),
        unique=True,
        nullable=False,
    )

    image_file: Mapped[str | None] = mapped_column(
        String(200),
        nullable=True,
        default=None,
    )

    posts: Mapped[list[Post]] = relationship(
        back_populates="author"
    )
```

## 10.1 Understanding __tablename__

```python
__tablename__ = "users"
```

This specifies the database table name.

## 10.2 Understanding String(50)

```python
username: Mapped[str] = mapped_column(String(50))
```

The database column is configured as a string column with a declared length of 50.

You should also validate the maximum length with Pydantic.

## 10.3 Understanding unique=True

```python
unique=True
```

This prevents duplicate values in the database column.

For example, two users should not have the same username.

Important: a Pydantic validator cannot establish whether a username already exists in the database.

You need a database query for an early check, and you must still handle database uniqueness errors because simultaneous requests can create race conditions.

## 10.4 Understanding nullable=False

```python
nullable=False
```

This tells the database that the column cannot contain SQL NULL.

It is different from Pydantic's required-field validation.

## 10.5 Understanding nullable=True

```python
image_file: Mapped[str | None] = mapped_column(
    String(200),
    nullable=True,
    default=None,
)
```

This column can contain a string or SQL NULL.

The Python annotation:

```python
str | None
```

means the Python attribute may be a string or `None`.

## 10.6 Understanding the image_path property

Your code:

```python
@property
def image_path(self) -> str:
    if self.image_file:
        return f"/media/profile_pics/{self.image_file}"

    return "/static/profile_pics/default.jpg"
```

This is a Python property.

It is not a database column.

When you access:

```python
user.image_path
```

Python executes the property and returns the calculated path.

For example:

```python
user.image_file = "profile.png"

print(user.image_path)
```

Output:

```text
/media/profile_pics/profile.png
```

If the user has no image:

```text
/static/profile_pics/default.jpg
```

This is useful when you want to calculate a value without storing it in the database.

---

# SECTION 11: Understanding the Post Model

```python
from datetime import UTC, datetime

class Post(Base):
    __tablename__ = "posts"

    id: Mapped[int] = mapped_column(
        Integer,
        primary_key=True,
    )

    title: Mapped[str] = mapped_column(
        String(100),
        nullable=False,
    )

    content: Mapped[str] = mapped_column(
        Text,
        nullable=False,
    )

    user_id: Mapped[int] = mapped_column(
        ForeignKey("users.id"),
        nullable=False,
        index=True,
    )

    date_posted: Mapped[datetime] = mapped_column(
        DateTime(timezone=True),
        default=lambda: datetime.now(UTC),
    )

    author: Mapped[User] = relationship(
        back_populates="posts"
    )
```

## 11.1 String vs Text

```python
title: Mapped[str] = mapped_column(String(100))
```

Suitable for a short title.

```python
content: Mapped[str] = mapped_column(Text)
```

Suitable for longer text.

## 11.2 Understanding DateTime

```python
DateTime(timezone=True)
```

Configures a datetime column with timezone support where supported by the database dialect.

Your Python default:

```python
default=lambda: datetime.now(UTC)
```

Generates a UTC-aware datetime when a new row is inserted without an explicit date.

Why use a lambda?

```python
default=datetime.now(UTC)
```

This evaluates the function immediately when the module loads.

But:

```python
default=lambda: datetime.now(UTC)
```

evaluates the function when SQLAlchemy needs the default value.

That means each inserted record can receive its own creation timestamp.

Note: SQLite does not preserve timezone information in the same way as PostgreSQL. Understand the behavior of your selected database and normalize timestamps consistently, commonly to UTC.

---

# SECTION 12: Understanding ForeignKey

This is one of the most important database concepts.

Your Post model contains:

```python
user_id: Mapped[int] = mapped_column(
    ForeignKey("users.id"),
    nullable=False,
)
```

What does it mean?

A post belongs to a user.

Imagine these tables.

### users

| id | username |
|---|---|
| 1 | alice |
| 2 | bob |

### posts

| id | title | user_id |
|---|---|---|
| 1 | First post | 1 |
| 2 | Second post | 1 |
| 3 | Third post | 2 |

The `user_id` column identifies the owner of each post.

The foreign key:

```python
ForeignKey("users.id")
```

means that `posts.user_id` references `users.id`.

The general syntax is:

```python
ForeignKey("table_name.column_name")
```

Examples:

```python
ForeignKey("users.id")
```

```python
ForeignKey("posts.id")
```

```python
ForeignKey("categories.id")
```

A foreign key helps maintain referential integrity.

---

# SECTION 13: Understanding relationship()

Your models contain:

```python
# User model
posts: Mapped[list[Post]] = relationship(
    back_populates="author"
)
```

And:

```python
# Post model
author: Mapped[User] = relationship(
    back_populates="posts"
)
```

These are ORM relationships.

They allow you to navigate between related Python objects.

For example:

```python
user = db.get(User, 1)

print(user.posts)
```

You can access the user's posts.

Or:

```python
post = db.get(Post, 1)

print(post.author)
```

You can access the author of a post.

## 13.1 ForeignKey vs relationship()

Do not confuse them.

| ForeignKey | relationship |
|---|---|
| Database-level reference | ORM-level navigation |
| Defines referential integrity | Defines object relationships |
| Usually maps to a database column | Does not create a separate database column |
| Example: `user_id` | Example: `post.author` |

A conventional one-to-many relationship usually needs both.

## 13.2 Understanding back_populates

```python
posts = relationship(back_populates="author")
```

And:

```python
author = relationship(back_populates="posts")
```

The names refer to the corresponding attribute on the other model.

This allows SQLAlchemy to coordinate both sides of the relationship.

For example:

```python
user = db.get(User, 1)

post = Post(
    title="My new post",
    content="Learning SQLAlchemy.",
    author=user,
)

db.add(post)
db.commit()
```

SQLAlchemy can use the relationship to determine the corresponding user ID.

You can also assign the foreign key directly:

```python
post = Post(
    title="Another post",
    content="Using user_id.",
    user_id=user.id,
)
```

---

# SECTION 14: Understanding from __future__ import annotations

At the beginning of your models.py:

```python
from __future__ import annotations
```

Your models refer to each other.

The User model references Post:

```python
posts: Mapped[list[Post]]
```

The Post model references User:

```python
author: Mapped[User]
```

The future annotations import postpones annotation evaluation.

This makes forward references easier to write.

Without it, you may need quoted references in certain arrangements:

```python
posts: Mapped[list["Post"]]
```

---

# SECTION 15: Understanding Pydantic Schemas

Now we reach the most important new concept.

You can use Pydantic in `schemas.py` to validate incoming data and define outgoing data.

Pydantic is not your database ORM.

It is responsible for:

- Data validation
- Parsing
- Type checking and conversion where permitted
- Serialization
- Defining API input and output structures

## 15.1 Why create schemas.py?

Imagine your database Post model:

```python
class Post(Base):
    __tablename__ = "posts"

    id: Mapped[int]
    title: Mapped[str]
    content: Mapped[str]
    user_id: Mapped[int]
    date_posted: Mapped[datetime]
```

When a client creates a post, should the client provide all these fields?

Usually not.

The database should generate the ID.

The server should generate the creation timestamp.

Therefore, your input schema should contain only the fields the client is allowed to provide.

---

# SECTION 16: Understanding BaseModel

Pydantic models inherit from:

```python
from pydantic import BaseModel
```

Example:

```python
class PostCreate(BaseModel):
    title: str
    content: str
    user_id: int
```

This creates a Pydantic model.

If FastAPI receives:

```json
{
    "title": "My first post",
    "content": "Learning FastAPI.",
    "user_id": 1
}
```

Pydantic validates the data and creates a Python object.

Your endpoint receives:

```python
post.title
post.content
post.user_id
```

Instead of working directly with an unvalidated dictionary.

---

# SECTION 17: Understanding Field()

Import:

```python
from pydantic import Field
```

You can define validation constraints:

```python
class PostCreate(BaseModel):
    title: str = Field(
        min_length=1,
        max_length=100,
    )

    content: str = Field(
        min_length=1,
    )

    user_id: int = Field(
        gt=0,
    )
```

Let's understand the arguments.

| Argument | Meaning |
|---|---|
| `min_length` | Minimum string or collection length |
| `max_length` | Maximum string or collection length |
| `gt` | Greater than |
| `ge` | Greater than or equal |
| `lt` | Less than |
| `le` | Less than or equal |
| `multiple_of` | Value must be a multiple of another number |
| `pattern` | Regular expression pattern |

Examples:

```python
age: int = Field(ge=18)
```

```python
price: float = Field(gt=0)
```

```python
username: str = Field(
    min_length=3,
    max_length=50,
)
```

```python
description: str = Field(max_length=500)
```

Remember: Pydantic validation does not replace database constraints.

---

# SECTION 18: Creating schemas.py

A good schemas.py for your blog:

```python
from datetime import datetime

from pydantic import (
    BaseModel,
    ConfigDict,
    EmailStr,
    Field,
)


class UserCreate(BaseModel):
    username: str = Field(
        min_length=3,
        max_length=50,
    )

    email: EmailStr


class UserResponse(BaseModel):
    model_config = ConfigDict(
        from_attributes=True,
    )

    id: int
    username: str
    email: EmailStr
    image_file: str | None
    image_path: str


class PostCreate(BaseModel):
    title: str = Field(
        min_length=1,
        max_length=100,
    )

    content: str = Field(
        min_length=1,
    )

    user_id: int = Field(
        gt=0,
    )


class PostUpdate(BaseModel):
    title: str | None = Field(
        default=None,
        min_length=1,
        max_length=100,
    )

    content: str | None = Field(
        default=None,
        min_length=1,
    )


class PostResponse(BaseModel):
    model_config = ConfigDict(
        from_attributes=True,
    )

    id: int
    title: str
    content: str
    user_id: int
    date_posted: datetime
```

If you use `EmailStr`, install:

```bash
uv add email-validator
```

---

# SECTION 19: Understanding ConfigDict and from_attributes

This is extremely important when using Pydantic with SQLAlchemy.

Consider:

```python
class PostResponse(BaseModel):
    model_config = ConfigDict(
        from_attributes=True,
    )

    id: int
    title: str
    content: str
```

Why do we need:

```python
from_attributes=True
```

A SQLAlchemy model is a Python object, not an ordinary dictionary.

For example:

```python
post = db.get(Post, 1)
```

You access its values using attributes:

```python
post.id
post.title
post.content
```

Pydantic needs to know that it can read attributes from this object.

That is what:

```python
ConfigDict(from_attributes=True)
```

does.

Conceptually, Pydantic can read the ORM object's attributes and build its own response model.

Without this configuration, Pydantic may expect dictionary-like input and fail to validate the ORM object.

### Pydantic v1 vs Pydantic v2

Older Pydantic examples use:

```python
class Config:
    orm_mode = True
```

In Pydantic v2, use:

```python
model_config = ConfigDict(
    from_attributes=True,
)
```

---

# SECTION 20: Create Schema vs Response Schema

This is a very important design decision.

Consider these two schemas.

### PostCreate

```python
class PostCreate(BaseModel):
    title: str
    content: str
    user_id: int
```

### PostResponse

```python
class PostResponse(BaseModel):
    model_config = ConfigDict(
        from_attributes=True,
    )

    id: int
    title: str
    content: str
    user_id: int
    date_posted: datetime
```

Why are they different?

When creating a post:

- The client supplies title.
- The client supplies content.
- The client supplies user ID.
- The database generates the post ID.
- The server/database generates the creation date.

When returning a post:

- The client needs the generated ID.
- The client needs the title.
- The client needs the content.
- The client needs the user ID.
- The client may need the creation date.

This is why separate schemas are useful.

You may create:

```python
UserCreate
UserUpdate
UserResponse

PostCreate
PostUpdate
PostResponse
```

Each schema represents a different API contract.

---

# SECTION 21: Understanding Optional Fields

In Pydantic v2, these are different:

```python
nickname: str | None
```

This field is required, but it accepts `None`.

```python
nickname: str | None = None
```

This field is optional and defaults to `None` if omitted.

For update schemas, the second form is often useful.

Example:

```python
class PostUpdate(BaseModel):
    title: str | None = None
    content: str | None = None
```

The client can send only the fields it wants to update.

However, decide how your API should treat an explicit `null` value. Should it clear the field or should it be rejected? Your update logic and schema should agree.

---

# SECTION 22: Connecting Pydantic to SQLAlchemy

Now let's understand how both systems work together.

Consider:

```python
@app.post(
    "/api/posts",
    response_model=PostResponse,
)
def create_post(
    post: PostCreate,
    db: DatabaseSession,
):
    ...
```

There are two important schemas here.

```python
post: PostCreate
```

This validates incoming request data.

```python
response_model=PostResponse
```

This defines the response format.

The request schema and response schema have different responsibilities.

The endpoint connects them.

---

# SECTION 23: Creating Database Tables

For a learning application:

```python
from contextlib import asynccontextmanager

from fastapi import FastAPI

from database import Base, engine
import models


@asynccontextmanager
async def lifespan(app: FastAPI):
    Base.metadata.create_all(bind=engine)

    yield


app = FastAPI(lifespan=lifespan)
```

Why import models?

SQLAlchemy needs to know which models have been registered in the metadata before `create_all()` can create their tables.

Important: use Alembic migrations in production. `create_all()` is for creating missing tables, not evolving existing database structures.

---

# SECTION 24: Querying Database Records

Import:

```python
from sqlalchemy import select
```

To get all posts:

```python
result = db.execute(
    select(models.Post)
)

posts = result.scalars().all()
```

Understand each part.

## select()

```python
select(models.Post)
```

Builds a SQLAlchemy SELECT statement.

## db.execute()

```python
db.execute(statement)
```

Executes the statement through the database session.

## scalars()

```python
result.scalars()
```

Extracts the selected ORM objects from the result rows.

## all()

```python
result.scalars().all()
```

Returns all the matching objects as a list.

A shorter form:

```python
posts = db.scalars(
    select(models.Post)
).all()
```

---

# SECTION 25: Retrieving One Post

Using `Session.get()`:

```python
post = db.get(models.Post, post_id)
```

This is useful when retrieving a record by its primary key.

Example:

```python
@app.get(
    "/api/posts/{post_id}",
    response_model=PostResponse,
)
def get_post(
    post_id: int,
    db: DatabaseSession,
):
    post = db.get(models.Post, post_id)

    if post is None:
        raise HTTPException(
            status_code=404,
            detail="Post not found",
        )

    return post
```

You can also use:

```python
result = db.execute(
    select(models.Post).where(
        models.Post.id == post_id
    )
)

post = result.scalars().first()
```

For a primary-key lookup, `db.get()` is usually simpler.

---

# SECTION 26: Creating a Post

Your original code:

```python
@app.post(
    "/api/posts",
    response_model=PostResponse,
    status_code=status.HTTP_201_CREATED,
)
def create_post(
    post: PostCreate,
    db: DatabaseSession,
):
    user = db.get(models.User, post.user_id)

    if user is None:
        raise HTTPException(
            status_code=404,
            detail="User not found",
        )

    new_post = models.Post(
        title=post.title,
        content=post.content,
        user_id=post.user_id,
    )

    db.add(new_post)
    db.commit()
    db.refresh(new_post)

    return new_post
```

Let's understand the workflow.

### Step 1: Pydantic validates the request

```python
post: PostCreate
```

FastAPI validates the incoming JSON.

### Step 2: Check that the user exists

```python
user = db.get(models.User, post.user_id)
```

Pydantic can verify that `user_id` is a positive integer.

It cannot verify that the user actually exists in the database.

That is why we query the database.

### Step 3: Create a SQLAlchemy object

```python
new_post = models.Post(
    title=post.title,
    content=post.content,
    user_id=post.user_id,
)
```

Notice that we are creating an instance of the SQLAlchemy model, not the Pydantic schema.

### Step 4: Add the object to the session

```python
db.add(new_post)
```

SQLAlchemy begins tracking the object.

### Step 5: Commit

```python
db.commit()
```

This commits the transaction.

### Step 6: Refresh

```python
db.refresh(new_post)
```

Reloads the object from the database.

This is useful for retrieving generated values such as the primary key.

### Step 7: Return the object

```python
return new_post
```

FastAPI uses `PostResponse` to validate and serialize the ORM object.

A safer transaction pattern:

```python
try:
    db.add(new_post)
    db.commit()
    db.refresh(new_post)

except Exception:
    db.rollback()
    raise
```

---

# SECTION 27: Retrieving Posts Belonging to a User

Your original endpoint:

```python
@app.get(
    "/api/users/{user_id}/posts",
    response_model=list[PostResponse],
)
def get_user_posts(
    user_id: int,
    db: DatabaseSession,
):
    user = db.get(models.User, user_id)

    if user is None:
        raise HTTPException(
            status_code=404,
            detail="User not found",
        )

    result = db.execute(
        select(models.Post).where(
            models.Post.user_id == user_id
        )
    )

    posts = result.scalars().all()

    return posts
```

Why check the user first?

You want to distinguish:

- A user who does not exist.
- A user who exists but has no posts.

For a nonexistent user, you return `404`.

For an existing user with no posts, you return:

```json
[]
```

That is a deliberate API design choice.

---

# SECTION 28: Querying Through Relationships

Because you defined:

```python
posts = relationship(back_populates="author")
```

You can do:

```python
user = db.get(models.User, 1)

print(user.posts)
```

You can access the author's posts directly.

Similarly:

```python
post = db.get(models.Post, 1)

print(post.author.username)
```

This is one of the benefits of ORM relationships.

However, relationship loading behavior matters.

SQLAlchemy may execute another database query when you access a relationship that has not yet been loaded.

---

# SECTION 29: Understanding Lazy Loading and Eager Loading

Imagine you retrieve 100 users:

```python
users = db.scalars(
    select(models.User)
).all()
```

Then:

```python
for user in users:
    print(user.posts)
```

Depending on the relationship loading configuration, SQLAlchemy may execute one query for the users and additional queries for each user's posts.

This is called the N+1 query problem.

You can use eager loading.

## 29.1 selectinload()

```python
from sqlalchemy.orm import selectinload

statement = (
    select(models.User)
    .options(
        selectinload(models.User.posts)
    )
)

users = db.scalars(statement).all()
```

This loads users and their related posts efficiently using a separate query for the related records.

## 29.2 joinedload()

```python
from sqlalchemy.orm import joinedload

statement = (
    select(models.User)
    .options(
        joinedload(models.User.posts)
    )
)

users = db.scalars(statement).unique().all()
```

For collection relationships, joined eager loading can duplicate parent rows in the SQL result. Use `.unique()` when consuming such ORM results.

---

# SECTION 30: Updating a Post

Create an update schema:

```python
class PostUpdate(BaseModel):
    title: str | None = Field(
        default=None,
        min_length=1,
        max_length=100,
    )

    content: str | None = Field(
        default=None,
        min_length=1,
    )
```

Now create an endpoint:

```python
@app.patch(
    "/api/posts/{post_id}",
    response_model=PostResponse,
)
def update_post(
    post_id: int,
    payload: PostUpdate,
    db: DatabaseSession,
):
    post = db.get(models.Post, post_id)

    if post is None:
        raise HTTPException(
            status_code=404,
            detail="Post not found",
        )

    changes = payload.model_dump(
        exclude_unset=True
    )

    for field, value in changes.items():
        setattr(post, field, value)

    try:
        db.commit()
        db.refresh(post)

    except Exception:
        db.rollback()
        raise

    return post
```

## Understanding model_dump()

```python
payload.model_dump(exclude_unset=True)
```

Converts a Pydantic model into a dictionary containing only the fields supplied by the client.

Suppose the client sends:

```json
{
    "title": "Updated title"
}
```

The result is:

```python
{
    "title": "Updated title"
}
```

The existing content remains unchanged.

Important: the example schema permits explicit `null`. If you do not want clients to clear a field or assign `None` to a non-null database column, reject explicit null values with an appropriate validation design.

---

# SECTION 31: Deleting a Post

```python
@app.delete("/api/posts/{post_id}")
def delete_post(
    post_id: int,
    db: DatabaseSession,
):
    post = db.get(models.Post, post_id)

    if post is None:
        raise HTTPException(
            status_code=404,
            detail="Post not found",
        )

    try:
        db.delete(post)
        db.commit()

    except Exception:
        db.rollback()
        raise

    return {
        "message": "Post deleted successfully"
    }
```

`db.delete(post)` marks the ORM object for deletion.

`db.commit()` commits the deletion.

Important: in an authenticated application, verify that the current user owns the post or has the necessary permission before deleting it.

---

# SECTION 32: Understanding HTML Routes

Your application also contains routes that return HTML pages.

For example:

```python
@app.get(
    "/posts",
    include_in_schema=False,
    name="posts",
)
def home(
    request: Request,
    db: DatabaseSession,
):
    result = db.execute(
        select(models.Post)
    )

    posts = result.scalars().all()

    return templates.TemplateResponse(
        request,
        "home.html",
        {
            "posts": posts,
            "title": "Home",
        },
    )
```

This is different from returning JSON.

An API route:

```python
@app.get("/api/posts")
```

normally returns JSON.

An HTML route:

```python
@app.get("/posts")
```

can return a rendered template.

## 32.1 Understanding TemplateResponse

```python
templates.TemplateResponse(
    request,
    "home.html",
    {
        "posts": posts,
        "title": "Home",
    },
)
```

The arguments:

- `request`: the incoming request.
- `"home.html"`: the template to render.
- Dictionary: data passed into the template.

In Jinja:

```html
<h1>{{ title }}</h1>

{% for post in posts %}
    <article>
        <h2>{{ post.title }}</h2>
        <p>{{ post.content }}</p>
    </article>
{% endfor %}
```

## 32.2 Understanding include_in_schema=False

```python
include_in_schema=False
```

This hides the route from the generated OpenAPI documentation.

It does not disable the route.

This is useful for HTML pages that you do not want to display as API operations in Swagger documentation.

## 32.3 Understanding route names

```python
@app.get("/", name="home")
```

You can reverse the route:

```python
request.url_for("home")
```

For parameterized routes:

```python
@app.get("/posts/{post_id}", name="post_page")
```

You can write:

```python
request.url_for(
    "post_page",
    post_id=10,
)
```

This builds a URL using the route name rather than manually concatenating strings.

---

# SECTION 33: Complete database.py

```python
from collections.abc import Generator

from sqlalchemy import create_engine
from sqlalchemy.orm import (
    DeclarativeBase,
    Session,
    sessionmaker,
)

SQLALCHEMY_DATABASE_URL = "sqlite:///./blog.db"

engine = create_engine(
    SQLALCHEMY_DATABASE_URL,
    connect_args={
        "check_same_thread": False
    },
)

SessionLocal = sessionmaker(
    bind=engine,
    autocommit=False,
    autoflush=False,
)


class Base(DeclarativeBase):
    pass


def get_db() -> Generator[Session, None, None]:
    with SessionLocal() as db:
        yield db
```

---

# SECTION 34: Complete models.py

```python
from __future__ import annotations

from datetime import UTC, datetime

from sqlalchemy import (
    DateTime,
    ForeignKey,
    Integer,
    String,
    Text,
)

from sqlalchemy.orm import (
    Mapped,
    mapped_column,
    relationship,
)

from database import Base


class User(Base):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(
        Integer,
        primary_key=True,
        index=True,
    )

    username: Mapped[str] = mapped_column(
        String(50),
        unique=True,
        nullable=False,
    )

    email: Mapped[str] = mapped_column(
        String(120),
        unique=True,
        nullable=False,
    )

    image_file: Mapped[str | None] = mapped_column(
        String(200),
        nullable=True,
        default=None,
    )

    posts: Mapped[list[Post]] = relationship(
        back_populates="author",
    )

    @property
    def image_path(self) -> str:
        if self.image_file:
            return f"/media/profile_pics/{self.image_file}"

        return "/static/profile_pics/default.jpg"


class Post(Base):
    __tablename__ = "posts"

    id: Mapped[int] = mapped_column(
        Integer,
        primary_key=True,
        index=True,
    )

    title: Mapped[str] = mapped_column(
        String(100),
        nullable=False,
    )

    content: Mapped[str] = mapped_column(
        Text,
        nullable=False,
    )

    user_id: Mapped[int] = mapped_column(
        ForeignKey("users.id"),
        nullable=False,
        index=True,
    )

    date_posted: Mapped[datetime] = mapped_column(
        DateTime(timezone=True),
        default=lambda: datetime.now(UTC),
    )

    author: Mapped[User] = relationship(
        back_populates="posts",
    )
```

---

# SECTION 35: Complete schemas.py

```python
from datetime import datetime

from pydantic import (
    BaseModel,
    ConfigDict,
    EmailStr,
    Field,
)


class UserCreate(BaseModel):
    username: str = Field(
        min_length=3,
        max_length=50,
    )

    email: EmailStr


class UserResponse(BaseModel):
    model_config = ConfigDict(
        from_attributes=True,
    )

    id: int
    username: str
    email: EmailStr
    image_file: str | None
    image_path: str


class PostCreate(BaseModel):
    title: str = Field(
        min_length=1,
        max_length=100,
    )

    content: str = Field(
        min_length=1,
    )

    user_id: int = Field(
        gt=0,
    )


class PostUpdate(BaseModel):
    title: str | None = Field(
        default=None,
        min_length=1,
        max_length=100,
    )

    content: str | None = Field(
        default=None,
        min_length=1,
    )


class PostResponse(BaseModel):
    model_config = ConfigDict(
        from_attributes=True,
    )

    id: int
    title: str
    content: str
    user_id: int
    date_posted: datetime
```

---

# SECTION 36: Complete main.py

```python
from contextlib import asynccontextmanager
from typing import Annotated

from fastapi import (
    Depends,
    FastAPI,
    HTTPException,
    status,
)

from sqlalchemy import select
from sqlalchemy.orm import Session

import models

from database import (
    Base,
    engine,
    get_db,
)

from schemas import (
    PostCreate,
    PostResponse,
)


@asynccontextmanager
async def lifespan(app: FastAPI):
    Base.metadata.create_all(bind=engine)

    yield


app = FastAPI(
    title="Blog API",
    lifespan=lifespan,
)


DatabaseSession = Annotated[
    Session,
    Depends(get_db),
]


@app.get(
    "/api/posts",
    response_model=list[PostResponse],
)
def get_posts(db: DatabaseSession):

    result = db.execute(
        select(models.Post)
    )

    posts = result.scalars().all()

    return posts


@app.get(
    "/api/posts/{post_id}",
    response_model=PostResponse,
)
def get_post(
    post_id: int,
    db: DatabaseSession,
):

    post = db.get(
        models.Post,
        post_id,
    )

    if post is None:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="Post not found",
        )

    return post


@app.get(
    "/api/users/{user_id}/posts",
    response_model=list[PostResponse],
)
def get_user_posts(
    user_id: int,
    db: DatabaseSession,
):

    user = db.get(
        models.User,
        user_id,
    )

    if user is None:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="User not found",
        )

    result = db.execute(
        select(models.Post).where(
            models.Post.user_id == user_id
        )
    )

    return result.scalars().all()


@app.post(
    "/api/posts",
    response_model=PostResponse,
    status_code=status.HTTP_201_CREATED,
)
def create_post(
    post: PostCreate,
    db: DatabaseSession,
):

    user = db.get(
        models.User,
        post.user_id,
    )

    if user is None:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="User not found",
        )

    new_post = models.Post(
        title=post.title,
        content=post.content,
        user_id=post.user_id,
    )

    try:
        db.add(new_post)

        db.commit()

        db.refresh(new_post)

    except Exception:
        db.rollback()
        raise

    return new_post
```

If you place these modules inside an `app/` package, make your imports package-consistent, for example:

```python
from app import models
from app.database import Base, engine, get_db
from app.schemas import PostCreate, PostResponse
```

Run from the project root:

```bash
uv run uvicorn app.main:app --reload
```

---

# SECTION 37: Testing Your API

Start your application:

```bash
uv run uvicorn main:app --reload
```

Open:

```text
http://127.0.0.1:8000/docs
```

FastAPI provides Swagger UI.

You can test your API directly.

## 37.1 Create a post

Request:

```json
{
    "title": "Learning FastAPI",
    "content": "I am learning SQLAlchemy and Pydantic.",
    "user_id": 1
}
```

## 37.2 Example response

```json
{
    "id": 1,
    "title": "Learning FastAPI",
    "content": "I am learning SQLAlchemy and Pydantic.",
    "user_id": 1,
    "date_posted": "2026-11-24T08:30:00"
}
```

The ID and timestamp are generated by the application/database.

## 37.3 Test invalid input

Try:

```json
{
    "title": "",
    "content": "Some content",
    "user_id": 1
}
```

Pydantic rejects the empty title because of:

```python
min_length=1
```

Try:

```json
{
    "title": "Test",
    "content": "Some content",
    "user_id": -5
}
```

Pydantic rejects the negative user ID because of:

```python
gt=0
```

Try a positive but nonexistent user ID.

Pydantic accepts the positive integer, but your endpoint returns `404` because that user does not exist.

This demonstrates the difference between validation and database lookup.

---

# SECTION 38: Important Production Considerations

## 38.1 Pydantic does not replace database constraints

For example:

```python
username: str = Field(
    min_length=3,
    max_length=50,
)
```

This does not guarantee username uniqueness.

You still need:

```python
unique=True
```

in your SQLAlchemy model.

You should also check for duplicates and handle `IntegrityError` when committing.

## 38.2 Do not expose sensitive ORM fields

Do not automatically return every database column.

For example, never expose password hashes simply because they exist on your User model.

Use response schemas to control which fields are returned.

## 38.3 Understand database transactions

A transaction groups database operations.

You normally use:

```python
db.add(user)
db.commit()
```

If an operation fails:

```python
db.rollback()
```

A session should not be shared between concurrent requests.

For background tasks, create a separate session inside the task.

## 38.4 Synchronous vs asynchronous SQLAlchemy

Your example uses:

```python
create_engine()
```

and:

```python
Session
```

These are synchronous SQLAlchemy APIs.

A normal FastAPI endpoint is suitable:

```python
@app.get("/api/posts")
def get_posts(db: DatabaseSession):
    ...
```

FastAPI runs normal synchronous path operations in a worker thread pool.

Do not change it to `async def` just because FastAPI supports asynchronous endpoints.

For fully asynchronous database access, use:

- `create_async_engine()`
- `AsyncSession`
- An async-compatible database driver
- `await` for database operations

---

# SECTION 39: SQLAlchemy vs Django ORM

Since you have worked with Django, this comparison will help.

| Django / DRF | FastAPI / SQLAlchemy / Pydantic |
|---|---|
| Django Model | SQLAlchemy ORM Model |
| `CharField` | `String` |
| `TextField` | `Text` |
| `IntegerField` | `Integer` |
| `DateTimeField` | `DateTime` |
| Django `ForeignKey` | SQLAlchemy `ForeignKey` |
| Django reverse relation | SQLAlchemy `relationship()` |
| DRF Serializer | Pydantic Schema |
| Serializer validation | Pydantic validation |
| Serializer representation | Pydantic response schema |
| Django ORM manager | SQLAlchemy Session |
| `objects.get()` | `Session.get()` |
| `objects.filter()` | `select().where()` |
| `save()` | `add()` and `commit()` |
| `transaction.atomic()` | SQLAlchemy transaction management |
| Django migrations | Alembic migrations |

Remember that these are conceptual comparisons. Django and SQLAlchemy have different APIs and session/transaction lifecycles.

---

# SECTION 40: Common Mistakes

## Mistake 1: Using Pydantic models as database tables

Wrong understanding:

```python
class PostCreate(BaseModel):
    title: str
```

This does not create a database table.

Correct separation:

```python
class Post(Base):
    __tablename__ = "posts"
```

The SQLAlchemy class represents the database table.

```python
class PostCreate(BaseModel):
    title: str
```

The Pydantic class validates input.

## Mistake 2: Forgetting from_attributes=True

Use:

```python
model_config = ConfigDict(
    from_attributes=True,
)
```

when returning SQLAlchemy ORM objects through Pydantic response schemas.

## Mistake 3: Confusing mapped_column() and Field()

SQLAlchemy:

```python
title: Mapped[str] = mapped_column(
    String(100),
    nullable=False,
)
```

Pydantic:

```python
title: str = Field(
    min_length=1,
    max_length=100,
)
```

They serve different purposes.

## Mistake 4: Confusing relationship and ForeignKey

`ForeignKey` creates a database-level reference.

`relationship` provides ORM navigation.

## Mistake 5: Forgetting commit()

```python
db.add(post)
```

does not mean the transaction has been committed.

Use:

```python
db.commit()
```

## Mistake 6: Assuming refresh() commits

It does not.

Use:

```python
db.commit()
db.refresh(post)
```

## Mistake 7: Expecting create_all() to alter existing tables

`create_all()` creates missing tables. It is not a migration tool.

Use Alembic for schema changes.

---

# SECTION 41: SQLAlchemy Query Reference

## Get one record by primary key

```python
user = db.get(models.User, user_id)
```

## Get all posts

```python
posts = db.scalars(
    select(models.Post)
).all()
```

## Filter posts

```python
posts = db.scalars(
    select(models.Post).where(
        models.Post.user_id == user_id
    )
).all()
```

## Multiple conditions

```python
posts = db.scalars(
    select(models.Post).where(
        models.Post.user_id == user_id,
        models.Post.title == "FastAPI",
    )
).all()
```

Multiple conditions passed to `where()` are combined with AND.

## Order by date

```python
from sqlalchemy import desc

posts = db.scalars(
    select(models.Post).order_by(
        desc(models.Post.date_posted)
    )
).all()
```

## Limit results

```python
posts = db.scalars(
    select(models.Post).limit(10)
).all()
```

## Count records

```python
from sqlalchemy import func, select

total = db.scalar(
    select(func.count()).select_from(
        models.Post
    )
)
```

## Delete

```python
post = db.get(models.Post, post_id)

if post is not None:
    db.delete(post)
    db.commit()
```

---

# SECTION 42: Exercises

Do these exercises to consolidate what you have learned.

## Exercise 1: Create a Comment model

A user can write many comments, and a post can have many comments.

Create:

```python
class Comment(Base):
    __tablename__ = "comments"
```

Include:

- `id`
- `content`
- `user_id`
- `post_id`
- `created_at`

Think about the foreign keys and relationships.

## Exercise 2: Create Comment schemas

Create:

```python
CommentCreate
CommentResponse
```

The create schema should validate the comment content and the IDs.

The response schema should include the generated ID and creation date.

## Exercise 3: Add validation

Require:

- Comment content: 1–2,000 characters.
- User ID: positive integer.
- Post ID: positive integer.

Then check that the referenced user and post exist.

## Exercise 4: Build endpoints

Implement:

```text
POST   /api/posts/{post_id}/comments
GET    /api/posts/{post_id}/comments
GET    /api/comments/{comment_id}
DELETE /api/comments/{comment_id}
```

Remember to verify ownership before deleting.

## Exercise 5: Add pagination

Implement:

```text
GET /api/posts?skip=0&limit=10
```

Use:

```python
from typing import Annotated
from fastapi import Query

skip: Annotated[int, Query(ge=0)] = 0

limit: Annotated[int, Query(ge=1, le=100)] = 10
```

Then:

```python
statement = (
    select(models.Post)
    .offset(skip)
    .limit(limit)
)

posts = db.scalars(statement).all()
```

---

# SECTION 43: Quick Reference

## Database engine

```python
engine = create_engine(DATABASE_URL)
```

## Session factory

```python
SessionLocal = sessionmaker(
    bind=engine,
    autocommit=False,
    autoflush=False,
)
```

## Base class

```python
class Base(DeclarativeBase):
    pass
```

## Database dependency

```python
def get_db():
    with SessionLocal() as db:
        yield db
```

## SQLAlchemy column

```python
id: Mapped[int] = mapped_column(
    Integer,
    primary_key=True,
)
```

## Foreign key

```python
user_id: Mapped[int] = mapped_column(
    ForeignKey("users.id"),
)
```

## ORM relationship

```python
posts: Mapped[list[Post]] = relationship(
    back_populates="author"
)
```

## Pydantic input schema

```python
class PostCreate(BaseModel):
    title: str = Field(
        min_length=1,
        max_length=100,
    )
```

## Pydantic response schema

```python
class PostResponse(BaseModel):
    model_config = ConfigDict(
        from_attributes=True,
    )

    id: int
    title: str
```

## Query

```python
posts = db.scalars(
    select(Post)
).all()
```

## Create

```python
db.add(post)
db.commit()
db.refresh(post)
```

## Update

```python
post.title = "New title"
db.commit()
db.refresh(post)
```

## Delete

```python
db.delete(post)
db.commit()
```

## Partial update

```python
changes = payload.model_dump(
    exclude_unset=True
)
```

---

# FINAL: The Most Important Concept to Remember

Keep these responsibilities separate:

1. **Pydantic schema:** Defines and validates the data entering and leaving your API.
2. **SQLAlchemy model:** Defines how data is represented in database tables.
3. **Database session:** Manages ORM operations and transactions.
4. **Endpoint:** Connects validated data to your application logic.
5. **Database constraints:** Enforce rules on stored records.

When creating a post, the process is:

```text
CLIENT SENDS JSON
       |
       v
PostCreate VALIDATES INPUT
       |
       v
FASTAPI ENDPOINT RECEIVES DATA
       |
       v
ENDPOINT CHECKS THAT USER EXISTS
       |
       v
SQLALCHEMY Post OBJECT IS CREATED
       |
       v
SESSION ADDS THE OBJECT
       |
       v
SESSION COMMITS THE TRANSACTION
       |
       v
PostResponse READS ORM ATTRIBUTES
       |
       v
FASTAPI RETURNS JSON
```

Once you understand this separation, you can build much larger FastAPI applications without mixing database mapping, request validation, and business logic into one class.

## Official Documentation

- FastAPI SQL databases: https://fastapi.tiangolo.com/tutorial/sql-databases/
- FastAPI dependencies: https://fastapi.tiangolo.com/tutorial/dependencies/
- FastAPI response models: https://fastapi.tiangolo.com/tutorial/response-model/
- Pydantic models: https://docs.pydantic.dev/latest/concepts/models/
- Pydantic fields: https://docs.pydantic.dev/latest/concepts/fields/
- Pydantic validators: https://docs.pydantic.dev/latest/concepts/validators/
- SQLAlchemy ORM quick start: https://docs.sqlalchemy.org/en/20/orm/quickstart.html
- SQLAlchemy relationships: https://docs.sqlalchemy.org/en/20/orm/basic_relationships.html
- SQLAlchemy sessions: https://docs.sqlalchemy.org/en/20/orm/session_basics.html
- Alembic migrations: https://alembic.sqlalchemy.org/en/latest/tutorial.html
