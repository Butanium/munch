[![Latest Version](https://img.shields.io/pypi/v/munch.svg)](https://pypi.python.org/pypi/munch/)
[![Supported Python versions](https://img.shields.io/pypi/pyversions/munch.svg)](https://pypi.python.org/pypi/munch/)
[![Downloads](https://img.shields.io/pypi/dm/munch.svg)](https://pypi.python.org/pypi/munch/)

munch
==========

Installation
-------------

```
pip install munch
```

Usage
-----

munch is a fork of David Schoonover's **Bunch** package, providing similar functionality. 99% of the work was done by him, and the fork was made mainly for lack of responsiveness for fixes and maintenance on the original code.

Munch is a dictionary that supports attribute-style access, a la JavaScript:

```python

>>> from munch import Munch
>>> b = Munch()
>>> b.hello = 'world'
>>> b.hello
'world'
>>> b['hello'] += "!"
>>> b.hello
'world!'
>>> b.foo = Munch(lol=True)
>>> b.foo.lol
True
>>> b.foo is b['foo']
True

```


Dictionary Methods
------------------

A Munch is a subclass of ``dict``; it supports all the methods a ``dict`` does:

```python

>>> list(b.keys())
['hello', 'foo']

```

Including ``update()``:

```python

>>> b.update({ 'ponies': 'are pretty!' }, hello=42)
>>> print(repr(b))
Munch({'hello': 42, 'foo': Munch({'lol': True}), 'ponies': 'are pretty!'})

```

As well as iteration:

```python

>>> [ (k,b[k]) for k in b ]
[('hello', 42), ('foo', Munch({'lol': True})), ('ponies', 'are pretty!')]

```

And "splats":

```python

>>> "The {knights} who say {ni}!".format(**Munch(knights='lolcats', ni='can haz'))
'The lolcats who say can haz!'

```


Serialization
-------------

Munches happily and transparently serialize to JSON and YAML.

```python

>>> b = Munch(foo=Munch(lol=True), hello=42, ponies='are pretty!')
>>> import json
>>> json.dumps(b)
'{"foo": {"lol": true}, "hello": 42, "ponies": "are pretty!"}'

```

If JSON support is present (``json`` or ``simplejson``), ``Munch`` will have a ``toJSON()`` method which returns the object as a JSON string.

If you have [PyYAML](http://pyyaml.org/wiki/PyYAML) installed, Munch attempts to register itself with the various YAML Representers so that Munches can be transparently dumped and loaded.

```python

>>> b = Munch(foo=Munch(lol=True), hello=42, ponies='are pretty!')
>>> import yaml
>>> yaml.dump(b)
'!munch.Munch\nfoo: !munch.Munch\n  lol: true\nhello: 42\nponies: are pretty!\n'
>>> yaml.safe_dump(b)
'foo:\n  lol: true\nhello: 42\nponies: are pretty!\n'

```

In addition, Munch instances will have a ``toYAML()`` method that returns the YAML string using ``yaml.safe_dump()``. This method also replaces ``__str__`` if present, as I find it far more readable. You can revert back to Python's default use of ``__repr__`` with a simple assignment: ``Munch.__str__ = Munch.__repr__``. The Munch class will also have a static method ``Munch.fromYAML()``, which loads a Munch out of a YAML string.

Finally, Munch converts easily and recursively to (``unmunchify()``, ``Munch.toDict()``) and from (``munchify()``, ``Munch.fromDict()``) a normal ``dict``, making it easy to cleanly serialize them in other formats.


Default Values
--------------

``DefaultMunch`` instances return a specific default value when an attribute is missing from the collection. Like ``collections.defaultdict``, the first argument is the value to use for missing keys:

```python

>>> from munch import DefaultMunch
>>> undefined = object()
>>> b = DefaultMunch(undefined, {'hello': 'world!'})
>>> b.hello
'world!'
>>> b.foo is undefined
True

```

``DefaultMunch.fromDict()`` also takes the ``default`` argument:

```python

>>> undefined = object()
>>> b = DefaultMunch.fromDict({'recursively': {'nested': 'value'}}, undefined)
>>> b.recursively.nested == 'value'
True
>>> b.recursively.foo is undefined
True

```

Or you can use ``DefaultFactoryMunch`` to specify a factory for generating missing attributes. The first argument is the factory:

```python

>>> from munch import DefaultFactoryMunch
>>> b = DefaultFactoryMunch(list, {'hello': 'world!'})
>>> b.hello
'world!'
>>> b.foo
[]
>>> b.bar.append('hello')
>>> b.bar
['hello']

```


Converting an Existing Dictionary to Munch
------------------------------------------

If you have an existing dictionary that you've computed and want to convert it into a Munch object, you can use the `munchify` function. This function recursively transforms a dictionary into a Munch object via copy. Here's an example:

```python
>>> b = munchify({'urmom': {'sez': {'what': 'what'}}})
>>> b.urmom.sez.what
'what'
```

The `munchify` function can handle intermediary dicts, lists, and tuples (as well as their subclasses). However, your mileage may vary with custom data types. Here's a more complex example:

```python
>>> b = munchify({ 'lol': ('cats', {'hah':'i win again'}),
...         'hello': [{'french':'salut', 'german':'hallo'}] })
>>> b.hello[0].french
'salut'
>>> b.lol[1].hah
'i win again'
```


Miscellaneous
-------------

* It is safe to ``import *`` from this module. You'll get: ``Munch``, ``DefaultMunch``, ``DefaultFactoryMunch``, ``munchify`` and ``unmunchify``.
* Ample Tests. Just run ``pip install tox && tox`` from the project root.

Feedback
--------

Open a ticket / fork the project on [GitHub](http://github.com/Infinidat/munch).
