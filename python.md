---
layout: page
title: Python
---

# General Tips

* Try to follow [PEP8](https://www.python.org/dev/peps/pep-0008/), except for line length. Use a linter (like [pylint](http://www.pylint.org/)) to enforce them automatically in your code.
* Use full names for variables where possible. For example, choose `job` over `j`, `something_longer` over `sl`, but not `this_variable_name_is_the_longest_name_evar_celery_man`.
* Functions and classes should have at least minimal docstrings. A good guide to write them can be found in [PEP 0257](https://www.python.org/dev/peps/pep-0257/).
* No [mutable default params](http://docs.python-guide.org/en/latest/writing/gotchas/#mutable-default-arguments) please! No `def foo(bar={'my': 'default'}):` and no `def foo(bar=MyClass()):` (assuming the `MyClass` instance is mutable)
* For methods exposed to the outside world, especially exposed methods that read lists of data from the database, default limits should always be applied to prevent accidental or intentional DOS attacks. Such an example can be found in `webapp/controllers/locations.py:get_locations(..)`. Before its latest rewrite, it did not apply any bounds on the `count` argument, causing severe load issues on web application servers.

# Syntax

## Rules to ease reviews

* Don't leave trailing white spaces.
* Add a new line char at the end of each file.
* Add trailing commas when enumerating.

``` python
from foo import (
    bar,
    baz,              # <--
)
# Or
d = {
    "key1": "value",
    "key2": 123,      # <--
}
```

## [Declaring exceptions](https://stackoverflow.com/questions/1319615/proper-way-to-declare-custom-exceptions-in-modern-python/26938914#26938914)
### The example

```
class MyAppValueError(ValueError):
    def __init__(self, foo, *args):
        # Special attribute you desire with your Error,
        # perhaps the value that caused the error?:
        self.foo = foo

        # Allow users initialize misc. arguments as any other builtin Error
        super(MyAppValueError, self).__init__(message, foo, *args)
```

There's no need to write your own `__str__` or `__repr__`. The builtin ones
are very nice.

### The example, expanded

**Name** your exceptions [in accordance with
PEP8](https://www.python.org/dev/peps/pep-0008/#exception-names), i.e. use
suffixes like `Error` or `Warning` where appropriate.

Exception classes you declare **should inherit** (either directly or indirectly)
from the `Exception` class.  If there's more that can be told about the nature
of the exceptional situation, pick a more specific exception class to inherit
from, from the hierarchy of [predefined exceptions](https://docs.python.org/2/library/exceptions.html#exception-hierarchy).

The last parameter of `__init__()` method **should be `*args`** to allow for
passing arbitrary number of arguments, which in turn are passed on to
`super(...).__init__(..., *args)`.  That way, the Liskov Substitution
Principle isn't violated between your exception class and the `Exception`
class.

**Avoid** passing human-readable messages to exception constructors, prefer
passing problematic variable values instead and naming the exception class
descriptively.  Rationale: Exceptions are entities belonging to the realm of
the code, not to that of the users.  Users aren't being shown raw exceptions,
they're being shown error messages which are an interpretation of caught
exceptions.   There are considerations like localization and formatting of the
message that need to be taken into account before showing a message to the
user.  Trying to address such considerations at the point in code at which the
exception is risen is simply impractical.  Therefore, put all the constituents
of a good error message as exception class fields instead, and leave putting
together the error message up to the handler of the exception.  How the error
message will be presented depends on where the code will be used: Web
application may need to present it differently than an API.

```
class MissingContactPhoneNumberError(Exception):
    def __init__(self, contact_uuid, *args):
	self.contact_uuid = contact_uuid
        super(MissingContactPhoneNumberError, self).__init__(self.message, contact_uuid, *args)
```

and then at the exception handling point in code:

```
try:
    [...]
except MissingContactPhoneNumberError as e:
    print "Contact lacks a phone number: {}".format(e.contact_uuid)
```

## Handling exceptions

When writing controllers which handle a lot of Exceptions, try to encapsulate
the logic within one `try..except`:

``` python
try:
    job = get_my_job(job_ref) # Raises some exception when job_ref isn't correct
    job.change_it_somehow('weee') # Raises some other exception just because
    job.delete('never wanted it anyways') # Raises another exception
except MyJobWasntFoundException as e: # Handle each Exception individually with a descriptive name
    log.info(e.message) # Potentially log as error if serious
except JobCantBeAltered as e:
    log.info(e.message)
except JobDeleteFailedException as e:
    log.info(e.message)
```


**Avoid** extracting text from exceptions messages &mdash; there's usually
some other way:

```
try:
    [...]
    DBSession.flush()  # tease out database integrity constraint errors
except IntegrityError as e:
    if "name" in e.message:
	raise CFSValidationError("DUPLICATE NAME",
				    "Duplicate name {!r}".format(cfd.name))
```

It has a potential to fail tests when a message text is changed.
