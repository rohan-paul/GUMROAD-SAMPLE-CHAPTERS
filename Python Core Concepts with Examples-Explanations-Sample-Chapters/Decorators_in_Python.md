## Decorators are probably one of THE MOST powerful feature of Python

Let's go over it quickly.

Decorators provide a super flexible way to modify or enhance the behavior of functions or methods without altering their code. At its core, a decorator is just a higher-order function, a function that takes one or more functions as arguments and returns a new function.

The fundamental theory behind decorators is the ability of Python functions to be first-class citizens, which means they can be passed around and used as arguments.

```python
def decorator_function(func):
    def wrapper():
        print("Something is happening before the function is called.")
        func()
        print("Something is happening after the function is called.")
    return wrapper

@decorator_function
def say_hello():
    print("Hello!")

say_hello()
```

When `say_hello()` is called, the output will be:

```
Something is happening before the function is called.
Hello!
Something is happening after the function is called.
```

---------------

👉 **Value Add**: The `@decorator_function` syntax is a more readable way of expressing `say_hello = decorator_function(say_hello)`. It allows us to "wrap" the behavior of the `say_hello` function with additional code from `decorator_function`, without modifying `say_hello` directly.

---------------------

👉 **Common Decorators in Real-life Python Projects**

1. **Logging**:
   It's quite common to have a decorator that logs the metadata or actual data about the function execution. This helps in debugging and monitoring.

```python
import logging
logging.basicConfig(level=logging.INFO)

def log_decorator(func):
    def wrapper(*args, **kwargs):
        logging.info(f"Running '{func.__name__}' with arguments {args} and keyword arguments {kwargs}")
        return func(*args, **kwargs)
    return wrapper

@log_decorator
def add(x, y):
    return x + y

add(5, 3)
# INFO:root:Running 'add' with arguments (5, 3) and keyword arguments {}
```

👉 **Value Add**: Automatically logs every time the `add` function is called, without having to add logging logic inside the `add` function itself.

--------------------

2. **Timing**:
   Timing decorators measure the time a function takes to execute, which is helpful for performance monitoring.

```python
import time

def timing_decorator(func):
    def wrapper(*args, **kwargs):
        start_time = time.time()
        result = func(*args, **kwargs)
        end_time = time.time()
        print(f"'{func.__name__}' took {end_time - start_time} seconds to execute")
        return result
    return wrapper

@timing_decorator
def long_running_task():
    time.sleep(2)

long_running_task()
#'long_running_task' took 2.003575325012207 seconds to execute
```

👉 **Value Add**: Without altering the `long_running_task` function, we can measure its execution time.

-------------------------

3. **Authentication/Authorization**:
   In web frameworks like Flask or Django, decorators can be used to restrict access to certain views based on user roles or authentication states.

```python
# Assuming a hypothetical web framework
current_user = {"is_authenticated": True, "role": "admin"}

def admin_required(func):
    def wrapper(*args, **kwargs):
        if not current_user["is_authenticated"] or current_user["role"] != "admin":
            raise PermissionError("Admin privileges are required.")
        return func(*args, **kwargs)
    return wrapper

@admin_required
def view_admin_dashboard():
    print("Admin dashboard viewed!")

view_admin_dashboard()
```

👉 **Value Add**: The decorator ensures that the `view_admin_dashboard` function can only be accessed by authenticated users with an admin role.

---------------------------

4. 👉  **Retry Mechanism**: In scenarios where a function might fail due to temporary issues (e.g., a network problem), a decorator can be used to retry the function a specified number of times before finally failing.

```python
import time

def retry_decorator(max_retries=3, delay=1):
    def decorator(func):
        def wrapper(*args, **kwargs):
            for _ in range(max_retries):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    print(f"Failure. Retrying in {delay} seconds...")
                    time.sleep(delay)
            raise e
        return wrapper
    return decorator

@retry_decorator(max_retries=5, delay=2)
def unreliable_function():
    if time.time() % 2:
        raise Exception("Failed due to some reason!")
    print("Function executed successfully!")

unreliable_function()
```

👉 **Value Add**: The decorator provides a mechanism to handle temporary failures, making the system more robust and resilient.

-------------------------------

5. 👉  **Caching/Memoization**: For functions that are computationally expensive, a decorator can be used to cache results, ensuring that subsequent calls with the same arguments return quickly.

```python
def memoize(func):
    cache = {}

    def wrapper(*args):
        if args in cache:
            return cache[args]
        else:
            result = func(*args)
            cache[args] = result
            return result
    return wrapper

@memoize
def expensive_computation(x, y):
    time.sleep(5)
    return x + y

expensive_computation(1, 2)  # Takes 5 seconds
expensive_computation(1, 2)  # Returns instantly due to caching
```

👉 **Value Add**: The decorator reduces the number of expensive computations by caching results, leading to faster execution for repeated calls with the same arguments.

-----------------------------

Examples of decorators being used for rate limiting - A simple but very useful example

In this example, we'll limit a function to be called no more than a certain number of times per second.

Let's break down the code:

1. The `rate_limited` function is a decorator factory. It takes as input the maximum number of times a function can be called per second (`max_per_second`). It calculates the minimum interval between calls (`min_interval`) from this input.

2. The `decorate` function is the actual decorator. It decorates/wraps the function with some rate limiting behavior.

3. Inside `decorate`, we define `rate_limited_function`. This function is what will be called instead of the original function. It measures the time elapsed since the last call, and if not enough time has passed, it waits the remaining time. It then calls the function and stores the current time.

4. The `wraps(func)` decorator is used to make the `rate_limited_function` look like `func` from the outside. It takes the name, module, and docstring from `func`.

5. When we use `@rate_limited(2)`, we're telling Python to limit this function to be called no more than twice per second. This is useful when interacting with external services that may have usage limits, or to prevent your own services from being overloaded.

Remember that this is a simple example and does not handle multiple threads or processes calling the function concurrently. A production-grade rate-limiter might use a more advanced algorithm, keep its state in a shared, thread-safe data structure, or even use a distributed system when multiple processes or machines need to be rate-limited together.

```python

import time
from functools import wraps

def rate_limited(max_per_second):
    min_interval = 1.0 / float(max_per_second)

    def decorate(func):
        last_time_called = [0.0]

        @wraps(func)
        def rate_limited_function(*args, **kwargs):
            elapsed = time.perf_counter() - last_time_called[0]
            left_to_wait = min_interval - elapsed

            if left_to_wait > 0:
                time.sleep(left_to_wait)

            ret = func(*args, **kwargs)
            last_time_called[0] = time.perf_counter()
            return ret

        return rate_limited_function
    return decorate

@rate_limited(2)  # 2 per second at most
def print_hello():
    print(f"Hello, World! Time: {time.perf_counter()}")

# Test the rate limit
for _ in range(10):
    print_hello()

```

----------------------------

## Example of a more exhaustive rate limiter decorator that uses Python's `threading` library to ensure thread-safety.

This version uses a `threading.Lock` to prevent race conditions. We'll use the token bucket algorithm for rate limiting, which allows for bursty calls up to a certain limit, and refills the bucket with tokens at a constant rate.


1. We define a `RateLimiter` class that will act as our decorator. It holds a number of "tokens" (representing the number of times the function can be called), the maximum rate at which the tokens refill, and the last time a function call was attempted.

2. The lock (`self.lock = threading.Lock()`) is used to ensure that only one thread at a time can execute the code within the `with self.lock:` block, which ensures thread-safety.

3. In the `wrapper` function, we calculate the number of tokens that have been refilled since the last function call (`self.tokens += elapsed * self.max_rate`). We limit the number of tokens to `max_rate`, simulating a "bucket" that fills up to a certain point.

4. If there are not enough tokens to call the function (i.e., `self.tokens < 1`), we wait until enough tokens have been refilled.

5. Once we've ensured that there are enough tokens, we "spend" a token (`self.tokens -= 1`) and call the function.

This code is thread-safe, but does not handle multiple processes. A more complex version might use multiprocessing primitives, or an external service like Redis, to handle rate limiting across multiple processes and machines.

However, implementing such an advanced and reliable rate-limiter from scratch can be quite challenging and error-prone. There are many excellent libraries available, like ratelimit, redislimiter, Flask-Limiter (for Flask applications), etc. which can do this job for you in a robust and efficient manner.

------------

When you run this code, you should see "Hello, World!" printed roughly two times per second, even though there are multiple threads trying to call print_hello() at once. The rate limiter ensures that, on average, no more than two calls are made per second.

Remember that the timing may not be perfectly even due to the nature of thread scheduling and other system operations. Also, sleeping a thread does not guarantee that the thread will resume exactly after the sleep time has passed. The actual delay may be longer due to other running threads and system factors.


```py
import time
import threading
from functools import wraps

class RateLimiter:
    def __init__(self, max_rate):
        self.tokens = max_rate
        self.max_rate = max_rate
        self.last = time.perf_counter()
        self.lock = threading.Lock()

    def __call__(self, func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            with self.lock:
                now = time.perf_counter()
                elapsed = now - self.last
                self.last = now
                self.tokens += elapsed * self.max_rate
                if self.tokens > self.max_rate:
                    self.tokens = self.max_rate
                if self.tokens < 1:
                    left_to_wait = (1 - self.tokens) / self.max_rate
                    time.sleep(left_to_wait)
                self.tokens -= 1
            return func(*args, **kwargs)
        return wrapper

@RateLimiter(2)  # 2 per second at most
def print_hello():
    print(f"Hello, World! Time: {time.perf_counter()}")

# Now, let's test this
threads = []
for _ in range(10):
    thread = threading.Thread(target=print_hello)
    threads.append(thread)
    thread.start()

for thread in threads:
    thread.join()


```

===========================

### In conclusion, decorators leverage Python's first-class function capabilities to add layers of logic before and/or after function execution. They provide an elegant mechanism to augment functions without modifying their core logic, making them a valuable tool in the Python programmer's toolkit.

----------------------------

All the above were examples of general user-defined decorators.

## But Python has many built-in decorators (like `@property`, `@staticmethod`, `@classmethod`)

========================

### A) `@property`

The `@property` decorator allows a method in a class to be accessed like an attribute rather than as a method. It's a way of defining getters for instance attributes in an object-oriented way.

Example:

```python
class Circle:
    def __init__(self, radius):
        self._radius = radius

    @property
    def radius(self):
        return self._radius

    @property
    def diameter(self):
        return self._radius * 2

c = Circle(5)
print(c.radius)    # 5
print(c.diameter)  # 10
```

👉 Another Example - `@property` and `@<property>.setter`

The `@property` decorator, along with its sibling `@<property>.setter`, is used in scenarios where you need to perform some extra processing or validation when getting or setting an attribute.

```python
class Employee:
    def __init__(self, first_name, last_name):
        self._first_name = first_name
        self._last_name = last_name

    @property
    def email(self):
        return f"{self._first_name}.{self._last_name}@company.com"

    @property
    def full_name(self):
        return f"{self._first_name} {self._last_name}"

    @full_name.setter
    def full_name(self, name):
        self._first_name, self._last_name = name.split(' ')

emp = Employee('John', 'Doe')
print(emp.email)       # John.Doe@company.com
print(emp.full_name)   # John Doe
emp.full_name = 'Jane Doe'
print(emp.email)       # Jane.Doe@company.com
```

In this example, the `full_name` property is defined to return a formatted string. The `email` property is defined based on the first and last names. When we set `full_name`, it automatically updates the first and last names, as well as the email.


--------------------

### B) `@staticmethod`

The `@staticmethod` decorator indicates that a method belongs to a class and not to any specific instance. It doesn't take `self` as its first parameter, and it can be called on a class rather than an instance.

Example:

```python
class Math:
    @staticmethod
    def add(x, y):
        return x + y

print(Math.add(3, 4))  # 7
```

👉 Another Example -`@staticmethod`

The `@staticmethod` decorator is used when you need a function that doesn't depend on the instance or class state. This could be a utility function that is logically connected to the class but doesn't need the class or instance data.

```python
class MathUtils:
    @staticmethod
    def calculate_percentage(value, total):
        return (value / total) * 100

percentage = MathUtils.calculate_percentage(50, 200)
print(percentage)  # 25.0
```

In this case, `calculate_percentage` doesn't depend on any state and doesn't modify any state - it's just a pure function. But we group it within `MathUtils` because it's conceptually related.

---------------------

### C) `@classmethod`

The `@classmethod` decorator indicates that a method belongs to a class, and not an instance. Instead of `self`, it takes `cls` as its first parameter, which refers to the class itself. It's useful when you want to have methods that inherently pertain to the class, and not necessarily to any specific instance.

Example:

```python
class MyClass:
    count = 0

    def __init__(self):
        MyClass.count += 1

    @classmethod
    def instance_count(cls):
        return cls.count

obj1 = MyClass()
obj2 = MyClass()
print(MyClass.instance_count())  # 2
```

👉 Another Example -`@classmethod`

The `@classmethod` decorator is used when you want to create an alternative constructor for a class.

```python
class Employee:
    def __init__(self, full_name, age):
        self.name = full_name
        self.age = age

    @classmethod
    def from_string(cls, name_and_age_str):
        full_name, age = name_and_age_str.split(',')
        return cls(full_name, int(age))

emp1 = Employee('John Doe', 25)
emp2 = Employee.from_string('Jane Doe,30')

print(emp1.name, emp1.age)  # John Doe 25
print(emp2.name, emp2.age)  # Jane Doe 30
```

In this case, we've defined an alternative constructor that allows creating an `Employee` object from a string. This is a useful pattern when you have multiple ways to create an object.

--------------------

### General User-Defined Decorators

The `memoize` decorator I provided earlier is an example of a custom, general-purpose decorator. It wraps functions to provide caching behavior, in this case to remember expensive computations and return the cached result if the same computation is called again.

The main differences between the built-in decorators (`@property`, `@staticmethod`, `@classmethod`) and general user-defined decorators like `memoize` are:

1. **Purpose**: Built-in decorators like `@property`, `@staticmethod`, and `@classmethod` are designed for specific, commonly-needed OOP functionalities in classes. In contrast, user-defined decorators can serve any purpose the programmer imagines, such as logging, timing, access control, etc.

2. **Usage**: The built-in decorators are specifically for methods inside classes. The `memoize` decorator, as given, can be used on any function (and could be adapted to work on methods too).

3. **Implementation**: While the mechanics of how decorators work (they receive a function/method and return a new function/method) are the same for both built-in and user-defined decorators, the actual things they do are different based on the specific decorator's intent.

In essence, the main distinction lies in their use cases and purposes.

------------------


