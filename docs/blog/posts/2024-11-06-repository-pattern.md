---
date: 2024-11-06
title: Repository Pattern
categories:
- Design Patterns
- Domain Driven Design
readtime: 10
---

# The Repository Pattern

Hey there, friend! Today, I want to talk to you about a design Pattern that's really useful in software development: the Repository Pattern.

## Introduction

So, what is the Repository Pattern? In a nutshell, it's a way to abstract away the data access layer in your application. Think of it like a library where you can store and retrieve data.

Imagine you're building a simple blog application. You have a list of posts, and you want to be able to add, remove, and retrieve posts from the database. Without the Repository Pattern, you might have code like this:


=== "Post"
    ```py title='models/post.py'
    from dataclasses import dataclass


    @dataclass
    class Post:
        id: str
        title: str
        content: str

    ```
=== "Repository"
    ```py title='repository.py'
    from models.post import Post


    class PostRepository:
        def __init__(self) -> None:
            self.posts: dict[str, str] = {}

        def add(self, post: Post) -> None:
            self.posts[post.id] = post

        def get(self, id: str) -> Post:
            try:
                return self.posts[id]
            except KeyError as e:
                raise Exception(f"Book with ID {id!r} not found") from e

    ```
This code is not that bad, but it has some problems...

For one, it's tightly coupled to the database (in this case, a dictionary of posts, but it could be a Postgres database and even a CSV file). If you wanted to switch to a different database, you'd have to rewrite a lot of code.

And what if you wanted to add some extra functionality, like caching or logging? You'd have to add it to the class, which could make it harder to maintain.

## Repository Pattern to our rescue

Here is where the Repository Pattern comes in. It is an abstraction layer between your application code and the data access layer. Basically, it provides a way to interact with the data without knowing how it's stored or retrieved.

### First step

I generally start with an abstract repository or interface, that will be used after to have our concrete implementations of accesing our data.

In Python, an Abstract Base Class (ABC) is like a blueprint for other classes. It defines methods (using the decorator `@abstractmethod`) that any subclass must implement, but it doesn't actually provide any working code itself. It’s a way to say, “Any class that inherits from me must have these methods.”

By the way, the abstract methods can have some functionality, if really needed.

Let's see some code:

```py title='abstract_post_repository.py'
from abc import ABC, abstractmethod

from models.post import Post


class AbstractPostRepository(ABC):

    @abstractmethod
    def add(self, post: Post) -> None:
        raise NotImplementedError

    @abstractmethod
    def get(self, id: str) -> Post:
        raise NotImplementedError
```

### What next?

Now, let's create some classes that actually do the work. We’ll make a simple in-memory repository that stores posts in a Python dictionary and a repository that will use a relational database.

=== "Post"
    ```py title='models/post.py'
    from dataclasses import dataclass
    
    from models.post import Post


    @dataclass
    class Post:
        id: str
        title: str
        content: str

    ```

=== "Fake Repository"
    ```py title='fake_post_repository.py'
    from models.post import Post


    class FakePostRepository(AbstractPostRepository):
        def __init__(self) -> None:
            self.posts: dict[str, str] = {}

        def add(self, post: Post) -> None:
            self.posts[post.id] = post

        def get(self, id: str) -> Post:
            try:
                return self.posts[id]
            except KeyError as e:
                raise Exception(f"Book with ID {id!r} not found") from e
    ```

=== "SQL Repository"
    ```py title='sql_post_repository.py'
    from models.post import Post


    class SqlPostRepository(AbstractPostRepository):
        def __init__(self, session) -> None:
            self._session = session

        def add(self, post: Post) -> None:
            # SQL code to add the post to the database
            pass

        def get(self, id: str) -> Post:
            # SQL code to retrieve the post from the database
            pass
    ```
In this example, the `FakePostRepository` class is a fake implementation of the Repository Pattern. It's used for testing purposes, and it simulates the behavior of a real Repository.

!!! tip
    Creating fake implementations for your abstractions is a great way to gather design feedback: if it's difficult to fake, the abstraction is likely too complex.

The `SqlPostRepository` class is the real deal. It's responsible for connecting to a relational database and implementing the same `add` and `get` methods. This design highlights how we can effectively decouple domain logic from infrastructure logic.

!!! info
    In the `SqlPostRepository` we can pass a `session` object that already handles the connection and transaction management. I ommited the SQL queries, but I think you will be able to figure them out. 

## Conclusion

By using the Repository Pattern, you can decouple your application code from the data access layer. This makes it easier to switch to a different database or add extra functionality.

And as we saw, it’s easy to make a fake version of the repository for unit testing, or to swap out different storage solutions, because we’ve fully decoupled the model from infrastructure concerns.
