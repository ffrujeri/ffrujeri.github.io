---
layout: archive
title: 'Python Ecosystem'
date: 2014-01-10
permalink: /posts/2014/01/python-ecosystem
excerpt: "Overview of the Python ecosystem and popular libraries."
tags:
  - python
  - libraries
---

## Libraries
Interesting Python libraries to explore:
```Python
import json
import math
import string
import re
import random
import sys
import traceback
import functools
from collections import OrderedDict

import numpy
import sortedcontainers

import appdirs
import attr
import basilica
import click
import configparser
import django
import eli5
import flask
import flask_migrate
import flask_sqlalchemy
import gensim
import ipywidgets
import keras
import matplotlib
import mysql.connector
import nltk
import notebook
import pandas
import pdpbox
import plotly
import psycopg2
import pymongo
import pytest
import requests
import scipy
import seaborn
import shap
import sklearn
import spacy
import sqlalchemy
import tensorflow
import timeout_decorator
import toml
import toolz
import tweepy
import xgboost
import ydata_profiling
```

## Frameworks

```Python
import tensorflow
import keras
import sklearn
import xgboost
import gensim
import spacy
import nltk
import flask
import django
import plotly
import seaborn
import shap
import eli5
import pdpbox
import ydata_profiling
import basilica
import tweepy
```

## Questions:

Your task is to implement a simple container of integer numbers. As a first step, consider implementing the following two operations:

```
ADD <value> should add the specified integer value to the container. Returns an empty string.

EXISTS <value> should check whether the specific integer value exists in the container. Returns "true" if the value exists, "false" otherwise.
The container is supposed to be empty at the beginning of execution.

Given a list of queries, return the list of answers for these queries. To pass to the next level you have to pass all tests at this level.
```
Example
```
queries = [
    ["ADD", "1"],
    ["ADD", "2"],
    ["ADD", "5"],
    ["ADD", "2"],
    ["EXISTS", "2"],
    ["EXISTS", "5"],
    ["EXISTS", "1"],
    ["EXISTS", "4"],
    ["EXISTS", "3"],
    ["EXISTS", "0"]
]
```
the output should be 
```
solution(queries) = ["", "", "", "", "true", "true", "true", "false", "false", "false"].
```

Explanation:

```
Add 1, 2, 5, 2 -> [1, 2, 5, 2]
Numbers 2, 5, 1 exist in the container
Numbers 4, 3, 0 don't exist in the container
```

Input/Output
```
[execution time limit] 4 seconds (py3)

[memory limit] 1 GB

[input] array.array.string queries
An array of queries. It is guaranteed that all queries consist only of operations described in the description, and these operations satisfy the format described in the description.
All integers represented as strings are from range [-100, 100].

Guaranteed constraints:
1 ≤ queries.length ≤ 100.

[output] array.string

Array consisting of operation results.
```

The next step is to support removal from the container:
```
REMOVE <value> should remove a single occurrence of the specified value from the container. If the value has multiple occurrences, only one of them should be removed.
Previous level functionality should remain the same. To pass to the next level you have to pass all tests at this level.

Given a list of queries, return the list of answers for these queries.
```

Example

```
queries = [
    ["ADD", "1"],
    ["ADD", "2"],
    ["ADD", "2"],
    ["ADD", "3"],
    ["EXISTS", "1"],
    ["EXISTS", "2"],
    ["EXISTS", "3"],
    ["REMOVE", "2"],
    ["REMOVE", "1"],
    ["EXISTS", "2"],
    ["EXISTS", "1"]
]
```
the output should be:
```
solution(queries) = ["", "", "", "", "true", "true", "true", "true", "true", "true", "false"].
```

Explanation:

```
Add 1, 2, 2, 3 -> [1, 2, 2, 3]
Numbers 1, 2, 3 exist in the container
Remove 2, 1 -> [2, 3]
Number 2 exists in the container, number 1 doesn't exist
Input/Output

[execution time limit] 4 seconds (py3)

[memory limit] 1 GB

[input] array.array.string queries

An array of queries. It is guaranteed that all queries consist only of operations described in the description, and these operations satisfy the format described in the description.
All integers represented as strings are from range [-100, 100].

Guaranteed constraints:
1 ≤ queries.length ≤ 100.

[output] array.string

Array consisting of operation results.
```

The next step is to support the operation for finding the nearest integer in the container greater than the provided one:

```
GET_NEXT <value> should return the minimal integer in the container that is strictly greater than the provided value. In case there is no such integer in the container, return empty string.
Previous levels functionality should remain the same. To pass to the next level you have to pass all tests at this level.

Given a list of queries, return the list of answers for these queries.
```

Example

```
queries = [
    ["ADD", "1"],
    ["ADD", "2"],
    ["ADD", "2"],
    ["ADD", "4"],
    ["GET_NEXT", "1"],
    ["GET_NEXT", "2"],
    ["GET_NEXT", "3"],
    ["GET_NEXT", "4"],
    ["REMOVE", "2"],
    ["GET_NEXT", "1"],
    ["GET_NEXT", "2"],
    ["GET_NEXT", "3"],
    ["GET_NEXT", "4"]
]
```
the output should be
```
solution(queries) = ["", "", "", "", "2", "4", "4", "", "true", "2", "4", "4", ""].
```

Explanation:

```
Add 1, 2, 2, 4 -> [1, 2, 2, 4]
Get Next 1 -> "2"
Get Next 2 -> "4"
Get Next 3 -> "4"
Get Next 4 -> ""
Remove 2 -> [1, 2, 4]
Get Next 1 -> "2"
Get Next 2 -> "4"
Get Next 3 -> "4"
Get Next 4 -> ""
```

Input/Output

```
[execution time limit] 4 seconds (py3)

[memory limit] 1 GB

[input] array.array.string queries

An array of queries. It is guaranteed that all queries consist only of operations described in the description, and these operations satisfy the format described in the description.
All integers represented as strings are from range [-100, 100].

Guaranteed constraints:
1 ≤ queries.length ≤ 100.

[output] array.string

Array consisting of operation results.
```

```Python
from collections import defaultdict
from sortedcontainers import SortedSet

def solution(queries):
    container = defaultdict(int)
    sorted_values = SortedSet()
    result = []
    
    for query in queries:
        operation, value = query[0], int(query[1])
        
        if operation == "ADD":
            if container[value] == 0:
                sorted_values.add(value)
            container[value] += 1
            result.append("")
        elif operation == "EXISTS":
            result.append("true" if container[value] > 0 else "false")
        elif operation == "REMOVE":
            if container[value] > 0:
                container[value] -= 1
                if container[value] == 0:
                    sorted_values.discard(value)
                result.append("true")
            else:
                result.append("false")
        elif operation == "GET_NEXT":
            idx = sorted_values.bisect_right(value)
            result.append(str(sorted_values[idx]) if idx < len(sorted_values) else "")
    
    return result
```

## References
- [Deep Learning with Python](https://www.manning.com/books/deep-learning-with-python)
