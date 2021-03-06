# pystream

> Java like Streams Api for Python

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

Python package for Java like Streams. pystreams are lazy evaluated -- That is, evaluation starts only when a terminal
operation is applied

## Installation

pip install this repo.
(Note: Incompatible with Python 2.x)

```sh
pip3 install git+https://github.com/zahash/pystream.git
```

(or)

```sh
pip install git+https://github.com/zahash/pystream.git
```

## Usage examples

map and filter

```Python
from pystream import Stream

result = Stream([1, 2, 3]) \
    .filter(lambda x: x > 1) \
    .map(lambda x: x * 2) \
    .collect(list)

# [4, 6]
```

reduce

```Python
from pystream import Stream
import operator

result = Stream([1, 2, 3]) \
    .filter(lambda x: x > 1) \
    .map(lambda x: x * 2) \
    .reduce(operator.add, initial=20)

# 30
```

```Python
from pystream import Stream

result = Stream([1, 2, 3]) \
    .filter(lambda x: x > 1) \
    .map(lambda x: x * 2) \
    .reduce(lambda x, y: x + y, initial=20)

# 30
```

first

```Python
from pystream import Stream

result = Stream([1, 2, 3]) \
    .map(lambda x: x * 2) \
    .filter(lambda x: x > 1) \
    .first()

# 2
```

foreach

```Python
from pystream import Stream

Stream([1, 2, 3]) \
    .map(lambda x: x * 2) \
    .filter(lambda x: x > 1) \
    .foreach(print)

# >>> 2
# >>> 4
# >>> 6
```

distinct

```Python
from pystream import Stream

result = Stream([3, 2, 3, 1, 3, 2, 2]) \
    .map(lambda x: x * 2) \
    .distinct() \
    .filter(lambda x: x > 1) \
    .collect(list)

# [6, 4, 2]
```

group with list

```Python
from pystream import Stream, Grouper

class Person:
    def __init__(self, name, age):
        self.name, self.age = name, age

people = [
    Person("jack", 20),
    Person("jack", 30),
    Person("jill", 25)
]

result = Stream(people) \
            .group(
                key_fn=lambda p: p.name,
                val_fn=lambda p: p.age,
                grouper=Grouper(list, grouper_fn=lambda l, item: l.append(item))
            )

# {
#   "jack": [20, 30],
#   "jill": [25]
# }
```

group with string

```Python
from pystream import Stream, Grouper

class Person:
    def __init__(self, name, age):
        self.name, self.age = name, age

people = [
    Person("jack", 20),
    Person("jack", 30),
    Person("jill", 25),
    Person("jack", 40)
]

result = Stream(people) \
            .group(
                key_fn=lambda p: p.name,
                val_fn=lambda p: p.age,
                grouper=Grouper(str, grouper_fn=lambda s, item: f"{s}, {item}" if s else f"{item}")
            )

# {
#   "jack": "20, 30, 40",
#   "jill": "25"
# }
```

sorted

```Python
from pystream import Stream

result = Stream([5, 4, 3, 2, 1]) \
            .map(lambda x: x - 1) \
            .sorted() \
            .filter(lambda x: x % 2 == 0) \
            .collect(list)

# [0, 2, 4]
```

limit

```Python
from pystream import Stream

result = Stream(range(100)) \
    .limit(10) \
    .collect(list)

# [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```

count

```Python
from pystream import Stream

result = Stream(['a', 'b', 'c']) \
            .count()

# 3
```

takewhile

```Python
from pystream import Stream

result = Stream([1, 2, 2, 4, 5, 3, 2, 3, 5]) \
            .takewhile(lambda x: x != 3) \
            .collect(list)

# [1, 2, 2, 4, 5]
```

dropwhile

```Python
from pystream import Stream

result = Stream([1, 2, 2, 4, 5, 3, 2, 3, 5]) \
            .dropwhile(lambda x: x != 3) \
            .collect(list)

# [3, 2, 3, 5]
```

allmatch

```Python
from pystream import Stream

result = Stream(["cat", "fat", "rat"]) \
            .allmatch(lambda x: "at" in x)

# True
```

```Python
from pystream import Stream

result = Stream(["cat", "dog", "rat"]) \
            .allmatch(lambda x: "at" in x)

# False
```

anymatch

```Python
from pystream import Stream

result = Stream(["cat", "dog", "rat"]) \
            .anymatch(lambda x: "at" in x)

# True
```

```Python
from pystream import Stream

result = Stream(["cat", "dog", "rat"]) \
            .anymatch(lambda x: "z" in x)

# False
```

error handling

```Python
from pystream import Stream
import logging

def handler(err):
    logging.error(err)

result = Stream(['a', 'b', 'c', 10, 'd']) \
    .map(lambda x: x.upper()) \
    .catch(handler) \
    .collect(list)

# >>> ERROR:root:'int' object has no attribute 'upper'
# result = ['A', 'B', 'C', 'D']
```

multiple error handling

```Python
from pystream import Stream
import logging

def handle_upper(err):
    logging.error(err)

def handle_index_out_of_bounds(err):
    logging.warning(err)

result = Stream(['ab', 'cd', 'e', 10, 'fg']) \
    .map(lambda x: x.upper()) \
    .catch(handle_upper) \
    .map(lambda x: x[1]) \
    .catch(handle_index_out_of_bounds) \
    .collect(list)

# >>> WARNING:root:string index out of range
# >>> ERROR:root:'int' object has no attribute 'upper'
# result = ['B', 'D', 'G']
```

replace error value

```Python
from pystream import Stream

def sqrt(x):
    if x < 0:
        raise ValueError(x)
    return x ** 0.5

def handler_neg_sqrt(err):
    (value,) = err.args
    return value * 1000

result = Stream([4, 9, -3, 5]) \
    .map(sqrt) \
    .catch(handler_neg_sqrt) \
    .collect(list)

# [2.0, 3.0, -3000, 2.23606797749979]
```

specific error handling

```Python
def err_fn(x):
    if x == 'a':
        raise ValueError(x)
    if x == 'b':
        raise KeyError(x)
    return x

def handle_value_err(err):
    (value,) = err.args
    return value * 2

def handle_key_err(err):
    (value,) = err.args
    return value * 4

result = Stream(['e', 'a', 'g', 'd', 'b']) \
    .map(err_fn) \
    .catch(handle_value_err, ValueError) \
    .catch(handle_key_err, KeyError) \
    .collect(list)

# ['e', 'aa', 'g', 'd', 'bbbb']
```

## Development setup

Clone this repo and install packages listed in requirements.txt

```sh
pip3 install -r requirements.txt
```

## Meta

M. Zahash ??? zahash.z@gmail.com

Distributed under the MIT license. See `LICENSE` for more information.

[https://github.com/zahash/](https://github.com/zahash/)

## Contributing

1. Fork it (<https://github.com/zahash/pystream/fork>)
2. Create your feature branch (`git checkout -b feature/fooBar`)
3. Commit your changes (`git commit -am 'Add some fooBar'`)
4. Push to the branch (`git push origin feature/fooBar`)
5. Create a new Pull Request
