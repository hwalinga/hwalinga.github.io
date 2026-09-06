---
title: "Hashing a Python List to Save in a Database"
summary: "Benchmarking hashing of lists and then decide what to use in a database"
tags: ["python", "postgresql"]
date: 2021-03-26
lastmod: 2026-09-06
draft: true
lightCode: true
---

For this blog post I write about how you should save the hash of a list in a database
using Python. This is a problem I came across during my side job.
Even if you know about databases and hashing it might be a bit more tricky than you think.
The solution is not as simple as `hash(tuple(li))`

## Hashing

Hashing is a powerful technique to reduce a complex data object into something much smaller.
Useful for a faster lookup or for obfuscation (in cryptography).

## Indexing in databases

Hashing is very useful in databases for quick lookup of the values saved in the database.
For most databases this is already built-in, named an index. However, for a
compound structure such as a list/array, this is not as easily built-in.
Postgresql has an array type, but it's default indexing is a b-tree that compares
on the first difference of the array[^1]. Not using the hash of the array.
There are more complex indexing available in postgresql for use of compound
structures, but they provide functionality ranging from partially matches
to finding individual elements, but not for pure equality[^2]. You really need
to make something yourself[^3], which can be tricky if you need to use
this in combination with something as Django.

[^1]: https://www.postgresql.org/docs/current/functions-array.html
[^2]: https://www.postgresql.org/docs/9.5/gin.html
[^3]: https://dba.stackexchange.com/a/62868/226708


## The pitfalls of Python's built-in `hash` function

You might think the solution to this is simple. Just save the result of
`hash(tuple(li))` as an integer! But there are multiple reasons why this
is not a good idea. The `hash` function is mostly designed for speed,
not for a robust hash function. One of the problem is that the `hash`
function is machine dependent. It maps to a range dependent on the
bit size of the machine it runs on. 32-bit machine create a 32-bit hash,
and 64-bit machines hash a 64-bit. On top of that, different sessions on the
same machine produce different hashes. That is not useful if you save that
to a DB. Lastly, there are some quirks with the hash function, which is
the problem of the hash of -1, which is -2. This means that `hash(-1) == hash(-2)`.
Which will also extend for compound data structures: `hash((-1, 2, 5)) == hash((-2, 2, 5))`.

## Using `hashlib`

Luckily for more advanced hashing, Python provides the `hashlib` module.
Using `hashlib` we can use the cryptographic hash algorithm MD5.
Not anymore save to use for security uses, but fine for these kind of uses.
Since we are using this function for non-security uses it is recommended to
pass `usedforsecurity=False` to the hash function to prevent it from
throwing an error on systems not capable to produce the hash safe.
So you can produce the hash as follows: `hashlib.md5(b'example', usedforsecurity=False).digest()`
The MD5 algorithm maps to the 128-bit which is also much bigger than the default
`hash` function, making it very unlikely you have collisions.

## Fastest way to hash

As you could see, the input to the hash function is a bytes string. There are
a few different way to construct a byte string from a list. Here I enumerate them
as found in [4]. I rewrote the code for use in Python3 and rerun for a new benchmark.
(As a lot has changed in perfomance compared to Python2.)

[4]: https://stackoverflow.com/a/20419128/8477066

### Hashing benchmark

```py {style=github}
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
"""
Created on Fri 26 Mar 2021 02:52:08 PM CET

@author hielke
"""

import sys
import timeit

setup = """
import array
import random
import hashlib
import marshal
import pickle
import struct

r = [random.randrange(1, 1000) for _ in range(0, 1000000)]
ra = array.array('h', r)   # create an array of shorts equivalent

def method1(r):
    p = pickle.dumps(r, -1)
    return hashlib.md5(p).hexdigest()

def method2(r):
    p = str(r)
    return hashlib.md5(str.encode(p)).hexdigest()

def method3(r):
    p = ','.join(map(str, r))
    return hashlib.md5(str.encode(p)).hexdigest()

def method4(r):
    fmt = '%dh' % len(r)
    buf = struct.pack(fmt, *r)
    return hashlib.md5(buf).hexdigest()

def method5(r):
    a = array.array('h', r)
    return hashlib.md5(a).hexdigest()

def method6(r):
    m = marshal.dumps(r)
    return hashlib.md5(m).hexdigest()

# using pre-built array...
def pb_method1(ra):
    p = pickle.dumps(ra, -1)
    return hashlib.md5(p).hexdigest()

def pb_method2(ra):
    p = str(ra)
    return hashlib.md5(str.encode(p)).hexdigest()

def pb_method3(ra):
    p = ','.join(map(str, ra))
    return hashlib.md5(str.encode(p)).hexdigest()

def pb_method4(ra):
    fmt = '%dh' % len(ra)
    buf = struct.pack(fmt, *ra)
    return hashlib.md5(buf).hexdigest()

def pb_method5(ra):
    return hashlib.md5(ra).hexdigest()

def pb_method6(ra):
    m = marshal.dumps(ra)
    return hashlib.md5(m).hexdigest()
"""

statements = {
    "pickle.dumps(r, -1)": "method1(r)",
    "str(r)": "method2(r)",
    "','.join(map(str, r))": "method3(r)",
    "struct.pack(fmt, *r)": "method4(r)",
    "array.array('h', r)": "method5(r)",
    "marshal.dumps(r)": "method6(r)",
    # versions using pre-built array...
    "pickle.dumps(ra, -1)": "pb_method1(ra)",
    "str(ra)": "pb_method2(ra)",
    "','.join(map(str, ra))": "pb_method3(ra)",
    "struct.pack(fmt, *ra)": "pb_method4(ra)",
    "ra (pre-built)": "pb_method5(ra)",
    "marshal.dumps(ra)": "pb_method6(ra)",
}

N = 10
R = 3

timings = [(
    idea,
    min(timeit.repeat(statements[idea], setup=setup, repeat=R, number=N)),
) for idea in statements]

longest = max(len(t[0]) for t in timings)  # length of longest name

print('fastest to slowest timings (Python {}.{}.{})\n'.format(*sys.version_info[:3]),
      '  ({:,d} calls, best of {:d})\n'.format(N, R))

ranked = sorted(timings, key=lambda t: t[1])  # sort by speed (fastest first)
for timing in ranked:
    print("{:>{width}} : {:.6f} secs, rel speed {rel:>8.6f}x".format(
          timing[0], timing[1], rel=timing[1] / ranked[0][1], width=longest))
```

### Results benchmark

```text {style=github}
fastest to slowest timings (Python 3.9.1)
   (10 calls, best of 3)

        ra (pre-built) : 0.034508 secs, rel speed 1.000000x
     marshal.dumps(ra) : 0.037991 secs, rel speed 1.100938x
  pickle.dumps(ra, -1) : 0.042954 secs, rel speed 1.244763x
   pickle.dumps(r, -1) : 0.304317 secs, rel speed 8.818704x
      marshal.dumps(r) : 0.335770 secs, rel speed 9.730185x
  struct.pack(fmt, *r) : 0.353308 secs, rel speed 10.238412x
   array.array('h', r) : 0.554542 secs, rel speed 16.069912x
 struct.pack(fmt, *ra) : 0.713129 secs, rel speed 20.665553x
                str(r) : 1.552162 secs, rel speed 44.979662x
               str(ra) : 2.037568 secs, rel speed 59.046095x
 ','.join(map(str, r)) : 3.598528 secs, rel speed 104.280693x
','.join(map(str, ra)) : 3.709283 secs, rel speed 107.490227x
```

### Interpretation

The fastest way to construct the hash is using `marshal` for array type (from the `array` module),
and `pickle` for the default list type. This is a surprising difference between
the Python2 results where `marshal` was much faster. This is especially good
news if you read on the documentation of `marshal` that specifies that it
is mostly for internal use and can change between Python versions
(and this not useful if you want to save the results).
Making the choice obvious that `pickle` is the best way to pre-process the list
before hashing.

## Saving to the DB

Now we have the 128-bit hash, we need to save it in the DB.
There isn't a 128-bit integer type for the database, but the correct type
here is UUID which you can find on most databases. Constructed in Python
using the `uuid` module. Making the complete expression:

```py {style=github}
uuid.UUID(bytes=hashlib.md5(pickle.dumps(li), usedforsecurity=False).digest())
```

If you use Django,
you can simply use `django.db.models.UUIDField` to save this value.
