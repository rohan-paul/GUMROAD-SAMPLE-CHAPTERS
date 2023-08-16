## Deep Copy vs. Shallow Copy in Python: A Deep Dive

When you’re duplicating data structures in Python, the depth of the copy matters. Let's dissect the intricacies of deep and shallow copies, and why they're crucial for advanced Python development.

👉 **Understanding the core difference**:

Shallow Copy: Only the outermost object is duplicated, while the inner objects are still references and not actual copies. The `copy` module's `copy()` function can achieve this.

Deep Copy: Both the outer and inner objects are duplicated, ensuring that no referenced objects from the source are left. This can be accomplished using the `copy` module's `deepcopy()` function.

👉 **Code Illustration**:

```python
import copy

# Shallow copy
original_list = [[1, 2, 3], [4, 5, 6]]
shallow_copied_list = copy.copy(original_list)
shallow_copied_list[0][0] = 999

# Deep copy
original_list_2 = [[1, 2, 3], [4, 5, 6]]
deep_copied_list = copy.deepcopy(original_list_2)
deep_copied_list[0][0] = 999

print(original_list)         # [[999, 2, 3], [4, 5, 6]]
print(original_list_2)      # [[1, 2, 3], [4, 5, 6]]
```

 Here, modifying the shallow-copied list changed the original list as well. But the deep-copied list remained isolated.

---------------

👉 **Behind the curtains**:

Python stores variables as references to objects. When doing a shallow copy, the outermost container gets a new reference, but the inner objects still point to the same references. In contrast, a deep copy ensures every object has a new reference, making the copied object totally independent.

👉 **Real-life Application - Config File Generation**:
Imagine a configuration template for a software package. You want to generate multiple configuration files from this template, with slight variations for different server environments, without altering the master template.

```python
# Original config template
config_template = {
    "database": {
        "user": "admin",
        "password": "password123"
    },
    "server": {
        "port": 8080
    }
}

# Deep copy the template for production adjustments
production_config = copy.deepcopy(config_template)
production_config["database"]["password"] = "prod_secure_pass"
production_config["server"]["port"] = 8000

# The master template remains unchanged
print(config_template["database"]["password"])  # password123
print(config_template["server"]["port"])       # 8080
```

👉 By using a deep copy, modifications for the production environment don't affect the original template. This ensures consistent and error-free configuration generation.

===============

## 👉🐍 Understanding the nuances between shallow and deep copying is crucial, especially in more complex systems or data structures.

Always consider the depth and complexity of the structures you're working with and choose the copying method that ensures the safety and integrity of your data. As we've seen, even in a language as high-level as Python, the devil is often in the details! 🎸🔥🐍

Here are some examples which can eazily raise bugs if you are not checking properly between shallow and deep copy

---------------

## 👉🐍   Mutable Objects within Immutable Containers:

Imagine you have a tuple containing lists. Since tuples are immutable, many may think that shallow copying them might be safe. Let's explore:

```python
import copy

original_tuple = ([1, 2, 3], [4, 5, 6])
shallow_copied_tuple = copy.copy(original_tuple)

shallow_copied_tuple[0].append(7)
print(original_tuple)
# Surprise! It outputs: ([1, 2, 3, 7], [4, 5, 6])

```

**Explanation**: While tuples are immutable, the contained lists are mutable. A shallow copy doesn't protect the inner mutable objects from being modified.

If you want to ensure that the `original_tuple` remains intact, there are several strategies you can employ, depending on the desired behavior:

👉 . **Use Deep Copy**: If you want to keep the structure (i.e., a tuple of lists) but want to ensure the `original_tuple` remains unchanged after creating a copied version, use `deepcopy`.

    ```python
    import copy
    original_tuple = ([1, 2, 3], [4, 5, 6])
    deep_copied_tuple = copy.deepcopy(original_tuple)
    deep_copied_tuple[0].append(7)
    print(original_tuple)  # Outputs: ([1, 2, 3], [4, 5, 6])
    ```

👉. **Explicit Copy of Inner Lists**: Manually create a new tuple containing copies of the inner lists. This approach gives more control over how deep the copy should go.

    ```python
    original_tuple = ([1, 2, 3], [4, 5, 6])
    copied_tuple = (original_tuple[0][:], original_tuple[1][:])
    copied_tuple[0].append(7)
    print(original_tuple)  # Outputs: ([1, 2, 3], [4, 5, 6])
    ```

👉 . **Wrap the Inner Lists with a Custom Immutable Wrapper**: If you want more granular control, you can create custom wrappers around the lists to make them immutable:

    ```python

    class ImmutableList:
        def __init__(self, data):
            self._data = list(data)

        def __getitem__(self, index):
            return self._data[index]

        def __len__(self):
            return len(self._data)

        # You can continue adding other dunder methods as necessary, but no mutating methods!

    original_tuple = (ImmutableList([1, 2, 3]), ImmutableList([4, 5, 6]))

    ```

With this approach, trying to modify the inner lists (e.g., using `append()`) will raise an error, unless you specifically implement and allow such operations in the `ImmutableList` class.

Choose the method that fits best with your application's requirements. If you don't need to modify the inner collections post-creation, the first method (converting inner lists to tuples) is the simplest. If you do need modifiability but want to protect the original data, using deep copy or manual copying will be more appropriate.

===============================


## 👉🐍  Cyclic References:

Objects in Python can reference themselves, leading to cyclic structures. Shallow copy can't handle this, but deep copy can.

```python
cyclic_list = [1, 2, 3]
cyclic_list.append(cyclic_list)

# Try to deep copy it
deep_copied_cyclic = copy.deepcopy(cyclic_list)
```

Deep copy manages this well, but if you tried to manually deep copy this structure, you'd end up with an infinite loop!

================================

## 👉🐍 Custom Objects and Their Internal State:

Consider custom objects. Shallow copy might not be enough to prevent unintentional modifications:

```python
class Person:
    def __init__(self, name, pets):
        self.name = name
        self.pets = pets

john = Person("John", ["cat", "dog"])
shallow_john = copy.copy(john)

shallow_john.pets.append("fish")
print(john.pets)  # Outputs: ['cat', 'dog', 'fish']
```

Even though `john` and `shallow_john` are different objects, their internal state is shared.

================================

## 👉🐍 Shared References:

Consider a scenario with shared references:

```python
shared_list = [8, 9, 10]
original_data = [1, 2, 3, shared_list]
shallow_copied_data = copy.copy(original_data)

shared_list.append(11)

print(shallow_copied_data[3])  # Outputs: [8, 9, 10, 11]
```

**Explanation**: Even though we modified the `shared_list` after creating the `shallow_copied_data`, the changes are reflected in the shallow copy because they reference the same list.


===============

👉 **Deep Copy vs. Shallow Copy in Python: A Case of Object Versioning**

One less-trodden path in Python development involves maintaining versions of mutable objects. Think of it as a simplified versioning system where you can make changes to an object while preserving its previous states. Deep copying plays a pivotal role in such scenarios.

**Scenario**: You're working on a graphics software that allows designers to manipulate shapes. For every change the designer makes, the software needs to store that version of the shape so the designer can revert to any previous state.

👉 **Setting up the Shape Class**:

First, let's design a simple shape class with some attributes.


```python
class Shape:
    def __init__(self, color, points):
        self.color = color
        self.points = points  # list of x,y coordinates
```

👉 **Versioning System**:

Here, we'll maintain versions using a list. When a change is made, we'll store the current state using a deep copy.

```python
import copy

class ShapeVersioning:
    def __init__(self, shape):
        self.versions = [copy.deepcopy(shape)]

    def modify_shape(self, new_shape):
        self.versions.append(copy.deepcopy(new_shape))

    def revert_to_version(self, version_number):
        return copy.deepcopy(self.versions[version_number])
```

👉 **Using the System**:

Let's assume a designer creates a triangle and then decides to modify it.

```python
triangle = Shape("red", [(0, 0), (0, 1), (1, 0)])

# Initialize versioning system
version_system = ShapeVersioning(triangle)

# Designer decides to change the color
triangle.color = "blue"
version_system.modify_shape(triangle)

# Designer modifies a point
triangle.points[2] = (1, 1)
version_system.modify_shape(triangle)

# Reverting to the original triangle
original_triangle = version_system.revert_to_version(0)
print(original_triangle.color)  # Outputs: red
print(original_triangle.points) # Outputs: [(0, 0), (0, 1), (1, 0)]
```
---------------------




----------------------

👉 **Diving Deeper**:
Here's why a deep copy is vital. When storing versions, any mutable attributes (like our `points` list) need to be entirely independent between versions. A shallow copy wouldn't suffice as modifying `points` in one version would inadvertently affect other versions, thus defeating the versioning purpose.

👉 **Takeaway**:
While versioning systems can get much more complex in real-world scenarios (like Git), this simplified approach showcases the utility of deep copying in preserving the integrity of object versions. It's a unique blend of design principles and Python's built-in capabilities to enable feature-rich applications.

👉 **Takeaway**:
Deep and shallow copies are fundamental for managing data in Python. While a shallow copy is quicker and uses less memory, it can lead to unintended data alterations due to shared references. A deep copy, although more resource-intensive, guarantees complete independence between the source and the copied data. For critical operations, understanding the distinction is vital. Always choose wisely based on the requirements of your application.


------------------


