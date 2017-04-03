---
layout: post
date: 2017-04-02 22:01:49
title: On Duck-Typing
published: true
---

**Note: The following is written with Python in mind, but the idea just as easily applies to something like JavaScript with a s/isinstance/instanceof/g, among other things.**

Let's say you're writing a database pooling service [^1] in Python, and have something like this:

```python
class MyDBPool(object):
  """
  A database connection pool.
  """
  def __init__(self):
    # Some initialization here

  def get_connection(self):
    """
    Return a connection to the database.
    """
    # Your implementation here

```

Unsurprisingly, somewhere within in the rest of your code it's used:

```python
def fetch_user(db_pool, user_id):
  conn = db_pool.get_connection()
  resp = conn.query("SELECT * FROM users WHERE id = ?", user_id)
  # ...
```

Like any object-oriented programmer worth their salt, you like to think in abstractions. A neat little package in Python's standard library lets you mark out your interface.

```python
# abc as in "Abstract Base Class"
from abc import ABCMeta
from abc import abstractmethod

#
# Let's define our interface...
#
class DBPool(object, metaclass=ABCMeta):
  def __init__(object):
    pass

  # This decorator says "throw an error if this isn't implemented"
  @abstractmethod
  def get_connection(self):
    """
    Return a connection to the database.
    """
    pass

#
# ...and separate it from our implementation
#
class MyDBPool(DBPool):
  # Your implementation here
```

"Hold on, this can be even safer!", you think. "We shouldn't just let you throw anything in when used, you have to *support the interface*."

```python
def fetch_user(db_pool, user_id):
  # Aha! Throw in a quick type-check
  if not isinstance(db_pool, DBPool):
    raise TypeError("db_pool does not implement DBPool")
  conn = db_pool.get_connection()
  resp = conn.query("SELECT * FROM users WHERE id = ?", user_id)
  # ...
```

Now you've fallen into a trap of types. Say it with me: **duck-typing has no explicit interfaces**. By doing this, you may think you've helped yourself but you've really just tied down your design to your implementation. Imagine someone else comes along and writes a better pooling solution, then you go to wire it in:

```python
db_pool = OnePoolToRuleThemAll(host, port)

fetch_user(db_pool, user_id)
# => TypeError: db_pool does not implement DBPool
```

Yet clearly, this new pool was written *for your interface*. When you're in a duck-typed language like Python, you need to stop thinking about types in terms of strict class hierarchy. If a class says it can do it, then let it. [^2]

Remember the mantra. If it walks like a duck, and quacks like a duck, it's (for all practical purposes) a duck.

[^1]: This is completely arbitrary, but helps me avoid the less concrete `Animal > Dog` sub-typing example that seems to be everywhere.

[^2]: By no means am I knocking the use of `abc`. It's a useful bookkeeping tool for keeping your own interfaces in check. Just don't fall into a trap of type-checking yourself, that's a job for the VM.
